---
title: (WPF) ICustomTypeDescriptor를 사용한 동적 바인딩
categories: WPF
key: 20221228_01
comments: true
tags: .NET WPF 동적바인딩 CustomTypeDescriptor PropertyDescriptor 바인딩 DynamicallyBinding
---

WPF 기본 Image컨트롤은 네트워크상 이미지 표시를 지원합니다.<br/>
```xml
<Image Source="url"/>
```

그런데 용량이 큰 고해상도 이미지인 경우 렌더링 과정이 동기 처리되기 때문에 UI가 멈추는 상황이 발생 합니다.<br/>
또한 로컬 파일 캐시 처리가 없기 때문에 프로그램 종료 후 해당 화면이 표시 될때는 다시 위와 같은 똑같은 상황이 발생 됩니다.<br/><br/>

이러한 상황을 해결하기 위해 이미지를 비동기로 다운받아서 표시 되도록 커스텀 컨트롤을 만들어 사용할 수 있습니다.<br/>
추가로 한번 받은 이미지는 더 이상 변경될 가능성이 없다면 로컬 파일로 저장해두었다가 로컬 파일 이미지를 load하도록 처리 되도록 하면 성능과 서버 트래픽 부하에 조금 나아질 것 입니다.<br/><br/>

다음은 커스텀 이미지 컨트롤 코드 입니다.<br/>
**[ImageHelper.cs]**<br/>
```cs
using System;
using System.IO;
using System.Net.Http;
using System.Threading.Tasks;
using System.Threading;
using System.Windows.Media.Imaging;

namespace AsyncImageControl;

public class ImageHelper
{
    public static BitmapImage? CreateBitmapImage(string? imgFullPath, int decodePixelWidth = 300)
    {
        if (string.IsNullOrWhiteSpace(imgFullPath)) return null;

        string imageFileName = Path.GetFileName(imgFullPath!);
        if (File.Exists($"{PathHelper.GetLocalDownloadDirectory()}{imageFileName}") == false)
            return null;

        BitmapImage bitmapImg = new BitmapImage();
        bitmapImg.BeginInit();
        bitmapImg.CacheOption = BitmapCacheOption.OnDemand;
        bitmapImg.CreateOptions = BitmapCreateOptions.DelayCreation;
        bitmapImg.DecodePixelWidth = decodePixelWidth;
        bitmapImg.UriSource = new Uri($"{PathHelper.GetLocalDownloadDirectory()}{imageFileName}");
        bitmapImg.EndInit();

        return bitmapImg;
    }

    /// <summary>
    /// 비동기로 Network 이미지를 다운로드하는 BitmapImage 생성
    /// </summary>
    /// <param name="imgFullPath">이미지 경로 (url)</param>
    /// <param name="isLocalDownloadFile">로컬 이미지 여부</param>
    /// <param name="decodePixelWidth">width 해상도</param>
    /// <returns></returns>
    public static async Task<BitmapImage?> CreateBitmapImage(string? imgFullPath, bool isLocalDownloadFile, int timeout, bool isFileCache = false, int decodePixelWidth = 300)
    {
        if(string.IsNullOrWhiteSpace(imgFullPath)) return null;

        BitmapImage? bitmapImg = null;
        if (isLocalDownloadFile)
        {
            bitmapImg = CreateBitmapImage(imgFullPath!, decodePixelWidth);
        }

        if (bitmapImg != null)
            return bitmapImg;

        // 로컬에 이미지가 없다면 이미지 다운로드
        using (HttpClient client = new HttpClient(new HttpClientHandler { MaxConnectionsPerServer = 10 }))
        {
            // 타임아웃 CancellationToken
            var tokenSource = new CancellationTokenSource(TimeSpan.FromSeconds(timeout));
            var token = tokenSource.Token;

            HttpResponseMessage response = await client.GetAsync(imgFullPath, token);
            byte[] bytes = await response.Content.ReadAsByteArrayAsync();

            bitmapImg = new BitmapImage();
            bitmapImg.BeginInit();
            bitmapImg.CacheOption = BitmapCacheOption.OnLoad;
            bitmapImg.DecodePixelWidth = decodePixelWidth;
            bitmapImg.StreamSource = new MemoryStream(bytes);
            bitmapImg.EndInit();

            if (isFileCache)
            {
                string imageFileName = Path.GetFileName(imgFullPath!);
                string savePath = $"{PathHelper.GetLocalDownloadDirectory()}{imageFileName}";
                File.WriteAllBytes(savePath, bytes);
            }
        }

        return bitmapImg;
    }
}

public class PathHelper
{
    public static string GetLocalDownloadDirectory()
    {
        return $"{System.AppDomain.CurrentDomain.BaseDirectory}\\LocalImages\\";
    }
}
```

**[AsyncNetworkImageControl.cs]**<br/>
```cs
using System;
using System.Windows.Media.Imaging;
using System.Windows;
using System.Windows.Controls;

namespace AsyncImageControl;

public class AsyncNetworkImageControl : Image
{
    public static readonly DependencyProperty UrlProperty =
            DependencyProperty.Register("Url", typeof(String), typeof(AsyncNetworkImageControl),
                new PropertyMetadata(new PropertyChangedCallback(OnUrlChanged)));

    public string Url
    {
        get { return (string)GetValue(UrlProperty); }
        set { SetValue(UrlProperty, value); }
    }

    public int Timeout
    {
        get; set;
    } = 15;

    public bool IsLocalImg
    {
        get; set;
    } = true;

    public bool IsFileCache
    {
        get; set;
    } = false;

    private static async void OnUrlChanged(DependencyObject property, DependencyPropertyChangedEventArgs e)
    {
        AsyncNetworkImageControl? asyncNetworkImageControl = property as AsyncNetworkImageControl;
        if (asyncNetworkImageControl == null) return;

        try
        {
            BitmapImage? bitmapImage = await ImageHelper.CreateBitmapImage(e.NewValue?.ToString(),
                asyncNetworkImageControl.IsLocalImg,
                asyncNetworkImageControl.Timeout,
                asyncNetworkImageControl.IsFileCache);
            asyncNetworkImageControl.Source = bitmapImage;
            bitmapImage?.Freeze();
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex);
        }
    }
}
```

**[사용 예]**<br/>
```xml
<Grid>
    <ScrollViewer>
        <StackPanel Orientation="Vertical">
            <local:AsyncNetworkImageControl
                Width="800"
                Height="800"
                IsLocalImg="True"
                IsFileCache="True"
                Url="http://arong.info:7004/articlesImage/test.bmp"/>
            
            <local:AsyncNetworkImageControl
                Width="800"
                Height="800"
                IsLocalImg="True"
                IsFileCache="True"
                Url="http://arong.info:7004/articlesImage/articles_1670828862799.jpg"/>
          
            <local:AsyncNetworkImageControl
                Width="500"
                Height="800"
                IsLocalImg="True"
                IsFileCache="True"
                Url="http://arong.info:7004/articlesImage/articles_1670819190396.jpg"/>
          
            <local:AsyncNetworkImageControl
                Width="500"
                Height="800"
                IsLocalImg="True"
                IsFileCache="True"
                Url="http://arong.info:7004/articlesImage/KakaoTalk_20221226_160032986.jpg"/>
          
            <local:AsyncNetworkImageControl
                Width="500"
                Height="800"
                IsLocalImg="True"
                IsFileCache="True"
                Url="http://arong.info:7004/articlesImage/KakaoTalk_20221226_155955866.jpg"/>
          
            <local:AsyncNetworkImageControl
                Width="500"
                Height="800"
                IsLocalImg="True"
                IsFileCache="True"
                Url="http://arong.info:7004/articlesImage/articles_1670918122944.jpg"/>
        </StackPanel>
    </ScrollViewer>
</Grid>
```


***

위 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
[Code_check - AsyncImageControl](https://github.com/tyeom/code_check/tree/main/TestSample/csharp/AsyncImageControl)



{% include content_adsense.html %}
