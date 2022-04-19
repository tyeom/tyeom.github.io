---
title: (WPF) MVVM패턴에서 ViewModel에서 팝업창 다루기
categories: WPF
key: 20220419_01
comments: true
tags: WPF MVVM 팝업 popup DialogService
---

MVVM패턴으로 설계되어 있는 프로젝트에서 새로운 팝업 윈도우를 띄우고자 하는 경우 어떻게 처리 해야 하는지 알아보도록 하겠습니다.<br/>
ViewModel과 상호 작용 없이 단순한 팝업 윈도우를 띄우는 것은 그냥 View의 코드비하인드 에서 단독으로 처리해도 상관 없지만<br/>
보통 이런 상황은 거의 없을 것 입니다.<br/>
특정 행위 이후 조건에 따라 -> 팝업 표시 같은 상황에 있어서 비지니스 로직은 Model에서 처리 되고 View 관련 로직 처리는 ViewModel에서 처리 하는 것이 맞습니다.

ViewModel에서 팝업을 띄우는 방법은 여러가지 방법이 있는데

1. Window를 사용하지 않고 Frame를 이용해 Visible 처리하는 방법
2. ViewModel에 IVew 추상화 인터페이스를 넘겨 처리하는 방법
3. 팝업 윈도우를 다룰 수 있는 서비스를 주입시켜 처리하는 방법

대략적으로 3가지 방법이 있습니다. **(Behavior 사용, 복잡한 방식으로 팝업 윈도우를 전용으로 관리하는 방법 등 여러가지 방법이 더 있습니다.)**

<!--more-->

그럼 하나씩 살펴 보겠습니다.

Frame을 이용한 팝업 처리
-

개인적으로는 이 방법이 현재 트렌드(?)에 맞는 팝업 처리지 않을까 생각 합니다.<br/>
현재 앱 스타일 추세에 따라 새로운 윈도우를 팝업으로 띄우지 않고 현재 컨텐츠 안에서 Popup컨트롤 또는 Frame컨트롤을 사용해서 팝업 내용을 표시하는 방법 입니다.

이렇게 처리 하는 경우 별도 작업 없이 Visible속성을 바인딩 해서 해당 뷰의 ViewModel 또는 메인 윈도우가 전체 컨텐츠를 감싸고 있는 껍데기 역할의 최상위 윈도우라면
Event Aggregator 를 사용해서 MainWindow ViewModel에서 해당 팝업 내용을 띄워 주도록 Visible 바인딩만 조작하면 됩니다.<br/>
그리고 modal로 처리되는 팝업이라면 별도로 뒷 부분 영역은 컨트롤 할 수 없도록 dim처리 시켜주기만 하면 됩니다.

**[View]**
```xml
<!--Popup시 백그라운드 처리-->
<Border Background="Black"
        Opacity="0.2">
  <Border.Style>
    <Style TargetType="Border">
      <Setter Property="Visibility" Value="Collapsed" />
      <Style.Triggers>
        <DataTrigger Binding="{Binding IsMainPopUpOpen}" Value="True">
          <Setter Property="Visibility" Value="Visible" />
        </DataTrigger>
      </Style.Triggers>
    </Style>
  </Border.Style>
  
  <!--Popup 백그라운드 처리 마우스 클릭시 팝업 Close-->
  <Border.InputBindings>
    <MouseBinding MouseAction="LeftClick" Command="{Binding PopUpCloseCommand}" />
  </Border.InputBindings>
</Border>
<!--Popup시 백그라운드 처리 END-->

<Frame x:Name="xPopupFrame"
               Visibility="{Binding IsMainPopUpOpen, Converter={StaticResource BoolToVisConverter}}"
               HorizontalAlignment="Center"
               VerticalAlignment="Center"
               Source="{Binding PopupPage}"
               NavigationUIVisibility="Hidden" />
```

이렇게 Frame에 팝업으로 표시할 컨텐츠를 바인딩 으로 표시하고 해당 Frame이 표시 될때 뒷 부분은 dim처리 하여 사용자 처리를 무효화 시킵니다.


IVew 추상화 인터페이스 사용
-

두번째로 팝업을 띄우는 행위를 추상화로 사용할 수 있는 인터페이스를 만들어서 ViewModel에서 해당 인터페이스를 받아 처리 하는 방식 입니다.<br/>
우선 모든 View의 인터페이스에서 사용 될 대표 IView인터페이스를 생성 합니다.<br/>
> **※ 이후 예제는 Microsoft.Toolkit.Mvvm 라이브러리를 사용합니다.**


**[IView.cs]**
```cs
public interface IView
{
}
```

이 예제에서는 단순 팝업에 대한 내용을 다루기 때문에 특별히 인터페이스에 노출 할 메서드는 없습니다.<br/>
IView 인터페이스를 가지고 각 View에 해당되는 별도의 인터페이스를 만들어서 해당 인터페이스에서 상속받아 사용될 예정 입니다.

메인 뷰에서 또다른 팝업 윈도우를 띄우기 위해 실제 팝업을 띄우는 메서드를 구현시킬 IMainView 인터페이스를 생성 합니다.<br/>
**[IMainView.cs]**
```cs
public interface IMainView : IView
{
  public bool? ShowPopupWindow();
}
```

그리고 ViewModel에서 IView를 넘겨 받을 수 있도록 ViewModelBase를 구현합니다.<br/>
**[ViewModelBase.cs]**
```cs
public abstract class ViewModelBase : ObservableObject
{
  public ViewModelBase(IView view)
  {
    View = view;
  }

  public IView View { get; private set; }
}

// 특정 IView 타입을 사용하는 경우 제네릭 ViewModelBase 사용
public abstract class ViewModelBase<T> : ObservableObject where T : IView
{
  public ViewModelBase(T view)
  {
    View = view;
  }
  
  public T View { get; private set; }
}
```

MainViewModel도 생성 합니다.<br/>
**[MainViewModel.cs]**
```cs
public class MainViewModel : ViewModelBase<IMainView>
{
  public MainViewModel(IMainView view) :
    base(view)
  {

  }
  
  private RelayCommand _popUpCommand;
  public RelayCommand PopUpCommand
  {
    get
    {
      return _popUpCommand ??
        (_popUpCommand = new RelayCommand(
            () =>
            {
              // 팝업 띄우기
              base.View.ShowPopupWindow();
            },
            null));
    }
  }
}
```

이제 MainWindow에서 해당 인터페이스를 구현합니다.<br/>
**[MainWindow.xaml.cs]**
```cs
public partial class MainWindow : Window, IMainView
{
  public MainWindow()
  {
    InitializeComponent();
    this.DataContext = new MainViewModel(this);
  }
  
  // 실제 팝업 표시 로직 구현
  public bool? ShowPopupWindow()
  {
    PopWindow popWindow = new();
    return popWindow.ShowDialog();
  }
}
```

이렇게 뷰의 행위가 추상화된 인터페이스를 뷰모델로 넘겨 받아 노출된 인터페이스 메서드를 사용하여 컨트롤 하는 구조 입니다.

DialogService 사용 방법
-

MVVM 패턴을 엄격히 지키고자 뷰의 코드비하인드 제로 코드를 paradigm으로 가져가는 경우 코드비하인드 처리를 깔끔하지 못할 수 있습니다.<br/>
이런 상황에서 심플하게 해결 할 수 있는 방법 중 외부에서 팝업을 대신 띄울 수 있게 처리하고 뷰 모델에서는 해당 처리자를 호출해서 사용하도록 구현할 수 있습니다.

팝업으로 사용되는 Popup Window는 실제 팝업 내용을 표시 하기 위한 역할의 껍데기고 실제 팝업 내용에 해당 되는 컨텐츠는 DateTemplate으로 미리 정의되어 있으며,
ContentControl에 바인딩되어 표시 되는 방식 입니다.<br/>
이 내용을 간략하게 도식화 해보면 다음과 같은 그림으로 설명할 수 있습니다.

![image](https://user-images.githubusercontent.com/13028129/163940942-903249dc-7f00-4405-9db1-0ccf823e0aad.png)

자 그럼 하나씩 구현해 보겠습니다.

먼저 Popup Window로 사용될 ViewModel을 구현 합니다. ViewModelBase은 위에서 구현했던 것을 그대로 사용하겠습니다.<br/>
**[PopViewModel.cs]**
```cs
public class PopViewModel : ViewModelBase
{
  private ViewModelBase _popupVM;
  
  public ViewModelBase PopupVM
  {
    get => _popupVM;
    set => this.SetProperty(ref _popupVM, value);
  }
}
```

그리고 Popup View를 구현 합니다.<br/>
**[PopWindow.xaml]**
```xml
<Window x:Class="WpfApp13.PopWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:WpfApp13"
        mc:Ignorable="d"
        Title="PopWindow" Height="450" Width="800">
    <Grid>
        <ContentControl Content="{Binding PopupVM}" />
    </Grid>
</Window>
```

**[PopWindow.xaml.cs]**
```cs
public partial class PopWindow : Window, IDialog
{
  public PopWindow()
  {
    this.DataContext = new PopViewModel();
    InitializeComponent();
  }
}
```

외부에 PopWindow 뷰를 띄우고 닫을 수 있는 메서드만 노출 시키기 위해 IDialog를 구현합니다.<br/>
**[IDialog.cs]**
```cs
public interface IDialog
{
  object DataContext { get; set; }

  void Show();

  bool? ShowDialog();

  void Close();
  
  // 추가로 필요한 기능은 여기에 해당 기능을 노출 하도록 추가 합니다.
}
```

그럼 IDialog를 컨트롤 하기 위한 IDialogService를 구현합니다.<br/>
**[IDialogService.cs]**
```cs
public interface IDialogService
{
  IDialog Dialog { get; }
  
  void SetVM(ViewModelBase vm);
}
```

**[DialogService.cs]**
```cs
public class DialogService : IDialogService
{
  private IDialog _popWindow;
  
  public DialogService()
  {
    _popWindow = App.Current.Services.GetService(typeof(PopWindow)) as IDialog;
  }
  
  public IDialog Dialog => _popWindow;
  
  public void SetVM(ViewModelBase vm)
  {
    if(_popWindow.DataContext is PopViewModel viewModel)
    {
      viewModel.PopupVM = vm;
    }
  }
}
```

PopWindow ViewModel의 PopupVM에 팝업으로 사용할 ViewModel을 적용함으로써 해당 ViewModel에 맞는 View가 PopWindow에 표시 됩니다.<br/>
그리고 IDialogService는 ViewModel에 의존성 주입 방식으로 사용할 수 있도록 다음과 같이 처리 합니다.

**[App.xaml.cs]**
```cs
public App()
{
  Services = ConfigureServices();
}

public new static App Current => (App)Application.Current;

public IServiceProvider Services { get; }

private static IServiceProvider ConfigureServices()
{
  ServiceCollection services = new ServiceCollection();
  // Services
  services.AddTransient<IDialogService, DialogService>();
  
  // Popup View
  services.AddTransient<PopWindow>();
  
  // Viewmodels
  services.AddTransient<MainViewModel>();
  return services.BuildServiceProvider();
}
```

이렇게 IoC에 IDialogService 를 등록하고 ViewModel에서 의존성 주입을 받아 사용할 수 있습니다.

**[MainWindow.xaml.cs]**
```cs
public MainWindow()
{
  InitializeComponent();
  this.DataContext = App.Current.Services.GetService(typeof(MainViewModel));
}
```

**[MainViewModel.cs]**
```cs
public class MainViewModel : ViewModelBase
{
  private readonly IDialogService _dialogService;
  public MainViewModel(IDialogService dialogService)
  {
    _dialogService = dialogService;
  }
  
  private RelayCommand _popUpCommand;
  public RelayCommand PopUpCommand
  {
    get
    {
      return _popUpCommand ??
        (_popUpCommand = new RelayCommand(
            () =>
            {
              // Popup1View 팝업 띄우기
              _dialogService.SetVM(new Popup1ViewModel());
              _dialogService.Dialog.ShowDialog();
            },
            null));
    }
  }
}
```

Popup1View 의 뷰와 ViewModel 매핑은 다음과 같이 DataTemplate으로 정의해 두었습니다.<br/>
**[App.xaml]**
```xml
<Application.Resources>
  <DataTemplate DataType="{x:Type local:Popup1ViewModel}">
    <local:Popup1/>
  </DataTemplate>
</Application.Resources>
```

**[Popup1.xaml]**
```xml
<UserControl x:Class="WpfApp13.Popup1"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" 
             xmlns:d="http://schemas.microsoft.com/expression/blend/2008" 
             xmlns:local="clr-namespace:WpfApp13"
             mc:Ignorable="d" 
             d:DesignHeight="450" d:DesignWidth="800">
    <Grid>
        <Button>팝업1 화면</Button>
    </Grid>
</UserControl>
```

**[Popup1ViewModel.cs]**
```cs
public class Popup1ViewModel : ViewModelBase
{
  // 필요한 Popup1ViewModel 구현
}
```

이렇게 MVVM패턴 사용시 ViewModel에서 팝업 띄우는 3가지 방식에 대해 알아보았습니다.

{% include content_adsense.html %}
