---
title: (WPF) WPF MVVM 패턴에 대해
categories: WPF
key: 20220121_01
comments: true
tags: WPF MVVM 아키텍처패턴
---

복잡한 프로그램일 수록 기본적인 설계단계에 있어 항상 다음과 같은 사항을 고려하지 않을 수 없습니다.<br/>
공통적 부분의 재사용성, 의존성 등 그리고 이런 고민은 어떻게 하면 효율적으로 사용자 인터페이스와 데이터를 시각적으로 연결 시켜주어야 할지 고민하기 마련 입니다.<br/>
이런 문제점을 해결하고자 많은 아키텍처 패턴들이 나와있습니다.<br/>
WPF는 그중에서 MVVM 아키텍처 패턴을 대해 완벽히 지원하고 권장하는 프레임워크 입니다.

<!--more-->

MVVM 아키텍처 패턴(이하 MVVM)은 사용자 인터페이스(뷰)영역과 비즈니스 로직을 분리시켜 의존성을 없애고 디자이너는 UX 중점으로 집중하고 개발자는 데이터 중심으로 접근 개발이 가능합니다.<br/>
(대부분 현실에서는 디자인 개발도 개발자 몫이긴 하겠지만..)<br/>
그럼 WPF에서 어떻게 MVVM을 적용해서 사용할 수 있는지, 그리고 현실적 타협(?)으로 최대한 MVVM패턴에 위배 되지 않게 어떻게 사용 할지에 대해 살펴보겠습니다.

MVVM 패턴이란
-

MVVM 패턴은 2005년 Microsoft에서 WPF 설계자중 한명인 존 구스만(John Gossman)이 발표했습니다.<br/>
MVVM 패턴은 뷰와 뷰모델간 서로 통보 처리를 위한 바인딩 기술이 핵심인데 WPF는 이러한 부분이 솔루션 스택에서 바인더를 제공합니다.<br/>
MVVM 패턴은 바로 바인딩을 통해 뷰 영역과 비즈니스 로직 및 데이터를 '분리' 시켜 설계하도록 하는 것이 목표 입니다.

**모델(Model)**<br/>
모델은 도메인 모델(데이터 엔티티, 엔티티의 속성 및 메서드)과 해당 되는 데이터를 처리하는 비즈니스 로직이 포함된 부분이라고 생각하면 됩니다.<br/>
가령 주문 시스템을 만든다고 하면 결제에 필요한 속성들과 데이터 조작 메서드 등, 주문 취소에 필요한 데이터 속성들 등

**뷰(View)**<br/>
사용자에게 보여지는 화면 입니다. 사용자에게 데이터들을 어떻게 표시하고 배치하는지에 대한 정보만 기술하고 있으며,<br/>
사용자와 상호 작용(마우스, 키보드 동작 등)은 바인딩을 통해 뷰 모델로 전달되어 집니다.

**뷰 모델(ViewModel)**<br/>
뷰에 대한 추상화 정보 입니다. 사용자의 상화작용에 대한 동작을 구현하고 그에 대한 데이터 처리 등을 모델을 통해 데이터를 가공할 수 있으며,<br/>
바인딩을 통해 다시 뷰에게 통보 역할을 합니다.

![image](https://user-images.githubusercontent.com/13028129/150717851-6dd20450-bd55-4527-84f1-a0a85160571f.png)<br/>
(이미지 출처 : https://ko.wikipedia.org/)

MVVM 패턴은 위 이미지 처럼 뷰와 데이터 처리를 분리 하고 데이터 처리 담당은 뷰모델과 모델로 전담 하여 의존성을 차단하고 있습니다.

뷰와 뷰모델간 바인딩
-

뷰와 뷰모델의 바인딩 처리 방식은 여러가지가 있을 수 있습니다. 여기서 바인딩 처리는 뷰에서의 데이터 바인딩 처리를 뜻하는 것이 아니고<br/>
뷰 <-- --> 뷰 모델 간의 연결 방식 처리를 말합니다.<br/>
쉽게 뷰에서 직접적으로 뷰모델을 연결할 수 있습니다. 가령 이런식으로<br/>
```cs
public MainWindow()
{
  InitializeComponent();
  this.DataContext = new MainViewmodel();
}
```

뷰의 코드비하인드에서 뷰모델 인스턴스를 생성하는 방법

또는<br/>
```xaml
<Window x:Class="WpfApp.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:WpfApp"
        mc:Ignorable="d">
    <Window.DataContext>
        <local:MainViewmodel/>
    </Window.DataContext>
    
    ... [생략] ...
```

이런식으로 xaml에서 직접 뷰모델 인스턴스를 생성하여 사용하는 방법

위 방법은 모두 뷰모델을 뷰에서 강력한 참조로 사용하고 있어 뷰와 뷰모델간 서로 의존성이 있지만 (개인적 생각으로는) 위 또한 MVVM 패턴을 강력히 위배 하는 행위는 아닙니다.<br/>
MS Docs 의 MVVM 패턴 내용이나 다른 MVVM 패턴 내용에 대해서도 '뷰가 직접적으로 뷰 모델을 참조해서는 안된다' 라는 내용은 없습니다.<br/>
하지만 분리 라는 중점을 보았을땐 확실히 좋은 방법은 아닙니다.

그럼 또 다른 방법은 무엇이 있을까요?<br/>
바로 뷰에서 직접 생성 및 참조 하지 않고 외부 즉 바깥쪽에서 뷰모델을 생성하여 설정해 주는 방법이 있습니다. 대략적인 코드로 표현 하면<br/>
```cs
public class ViewModelProvider : INotifyPropertyChanged
{
  private BaseViewModel _viewModel
  public BaseViewModel ViewModel
  { 
			get => _viewModel;
			set
      {
        _viewModel = value;
        OnPropertyChanged();
      }
		}
    
  #region INotifyPropertyChanged Members
  public event PropertyChangedEventHandler PropertyChanged;
  
  public void OnPropertyChanged([CallerMemberName] string propertyName = null)
  {
    PropertyChangedEventHandler handler = this.PropertyChanged;
    
    if (handler != null)
    {
      handler(this, new PropertyChangedEventArgs(propertyName));
    }
  }
  #endregion
}
```

**[App.xaml.cs]**
```cs
protected override void OnStartup(StartupEventArgs e)
{
  ViewModelProvider viewModelProvider = new ViewModelProvider();
  viewModelProvider.ViewModel = new MainWindowViewModel();
  
  MainWindow window = new MainWindow();
  window.DataContext = viewModelProvider.ViewModel;
  mainWindow.Show();
}
```

위 코드와 같이 뷰모델 생성을 담당하는 외부 ViewModelProvider가 뷰를 생성하면서 DataContext를 설정하는 방식 입니다.<br/>
뷰가 변경 될때 ViewModelProvider의 ViewModel속성만 변경해주면 됩니다.<br/>
위와 같은 방법은 뷰와 뷰 모델 간 강력한 참조를 없애고 서로간 분리가 되어 한쪽에 문제가 발생 되어도 프로그램 빌드나 유닛 테스트 자체는 실행 가능합니다.

뷰 전환 방법 1
-

이때 뷰의 변경은 ContentControl에 바인딩을 해두고 각 뷰 모델 타입에 대한 DataTemplate을 정의해서 뷰를 변경할 수 있습니다.<br/>
즉 ViewModelProvider의 ViewModel속성 하나로 해당 속성에 뷰 모델 인스턴스를 교체해 가면서 뷰를 변경하는 것 입니다.<br/>
**[MainWindow.xaml]**
```xaml
<Window x:Class="WpfApp.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:WpfApp"
        mc:Ignorable="d">
        <Grid>
          <ContentControl Content="{Binding .}" />
        </Grid>
</Window>
```

**[App.xaml]**
```xaml
<Application x:Class="WpfApp.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:views="clr-namespace:WpfApp.Views"
             xmlns:viewModels="clr-namespace:WpfApp.ViewModels">
    <Application.Resources>
      <DataTemplate DataType="{x:Type viewModels:뷰1ViewModel}">
		    <views:뷰1/>
	    </DataTemplate>
      
      <DataTemplate DataType="{x:Type viewModels:뷰2ViewModel}">
		    <views:뷰2/>
	    </DataTemplate>
      
      
      ... [View DataTemplate 3] ...
    </Application.Resources>
</Application>
```

이렇게 각 뷰에 대한 DataTemplate 을 각각 정의해 두고 DataTemplate의 DataType은 해당 뷰에 맞는 뷰 모델 타입을 설정하면<br/>
해당 ContentControl의 Content가 바인딩되어 있는 경우 **<span style="color: rgb(107, 173, 222);">System.Windows.FrameworkElement</span>** 클래스의 ApplyTemplate() 메서드로 인해서 해당 뷰 리소스, App 프로그램의 리소스에 접근해서 바인딩 타입에 맞는 DataTemplate 을 적용 해 줍니다.<br/>
```cs
viewModelProvider.ViewModel = new 뷰2ViewModel()
```
viewModelProvider에 바인딩 되어 있는 ViewModel에 뷰 모델 교체시 해당 타입에 맞는 DataTemplate을 찾아 ContentControl 에 적용됩니다.

하지만 이 방법은 단점이 있습니다.<br/>
뷰와 뷰 모델간 N:1 매칭이 안됩니다. 디자인이 다른 뷰 여러개와 뷰 모델 하나의 설정이 힘듭니다.<br/>
하나의 뷰에서 일뷰 디자인이 상황에 따라 변경 처리가 되어야 하는 경우는 **<span style="color: rgb(107, 173, 222);">System.Windows.Controls.DataTemplateSelector</span>** 을 사용해서<br/>
처리해도 되지만 화면 디자인중 여러 부분이 다르고 다소 복잡한 화면이라면 차라리 뷰를 따로 디자인해서 나누고 하나의 뷰 모델을 사용하는 것이 더 효과적일 수 있습니다.<br/>

DataTemplate의 DataType 내부적으로 리소스의 Key로 관리 되기 때문에 하나의 DataType만 정의할 수 있습니다.
![image](https://user-images.githubusercontent.com/13028129/150730440-665a265e-ac49-4402-90b7-5f93f7064591.png)<br/>
(DataTemplate의 DataType이 여러개 정의 되어 있을때 런타임시 익셉션 발생)

그리고 뷰가 표시 될때 무조건 재 생성 되어 렌더링 처리 됩니다. 이 부분은 추가 작접을 통해 캐시 처리 등을 구현해야 합니다.


뷰 전환 방법 2
-

위에서는 뷰 모델을 교체하면 자동으로 해당 뷰 모델에 맞는 뷰가 표시 되도록 DataTemplate을 사용하는 방법으로 뷰 전환에 대해 설명했습니다.<br/>
그리고 이 방법에 대한 단점을 설명했습니다. 

# 계속해서 작성중...

