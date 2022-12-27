---
title: (WPF) ICustomTypeDescriptor를 사용한 동적 바인딩
categories: WPF
key: 20221227_01
comments: true
tags: .NET WPF 동적바인딩 CustomTypeDescriptor PropertyDescriptor 바인딩 DynamicallyBinding
---

WPF에서 데이터 바인딩을 통해 개체를 참조하는 방식을 크게 세 가지 방식으로 제공하고 있습니다.<br/>
첫 번째는 이 글에서 알아볼 **<span style="color: rgb(107, 173, 222);">System.ComponentModel.CustomTypeDescriptor</span>** 와 **<span style="color: rgb(107, 173, 222);">System.ComponentModel.INotifyPropertyChanged</span>** 를 구현하여 
리플렉션을 이용해 속성 특성을 검색하고 변경 알림을 사용하는 방법이고<br/><br/>

두 번째는 **<span style="color: rgb(107, 173, 222);">System.ComponentModel.INotifyPropertyChanged</span>** 구현으로 특정 속성의 변경 알림만 구현해 주면 데이터 바인딩 엔진 자체에서 리플렉션을 사용하고 
필요한 속성을 참조합니다.<br/><br/>

세 번째는 **<span style="color: rgb(107, 173, 222);">System.Windows.DependencyProperty</span>** 를 통한 바인딩 방식 입니다.<br/>
이 경우 데이터 바인딩 엔진은 리플렉션을 사용하지 않기 때문에 속도가 가장 빠릅니다.

<!--more-->

> 이 글에서 다루는 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
> [Code_check - CustomTypeDescriptorEx](https://github.com/tyeom/code_check/tree/main/TestSample/csharp/CustomTypeDescriptorEx)

이번 글에서는 **<span style="color: rgb(107, 173, 222);">System.ComponentModel.CustomTypeDescriptor</span>** 클래스를 사용해 런타임시 동적으로 속성 바인딩 하는 방법을 알아 봅니다.<br/>
바인딩 속성이 뷰 포트에 보여질때 리플렉션으로 해당 속성을 검색하고 접근하기 때문에 **<u>성능면에서는 좋지 않기 때문에 사실 이런 방식의 바인딩이 필요할지 의문이지만 이러한 방법으로 동적 바인딩 
처리가 가능하다는 것만 알면 좋을 것 같습니다.</u>**<br/><br/>

WPF는 데이터 바인딩의 소스가 **<span style="color: rgb(107, 173, 222);">System.ComponentModel.ICustomTypeDescriptor</span>** 인터페이스가 구현 되어 있는 경우 바인딩에 노출하는 속성 특성 컬렉션 **<span style="color: rgb(107, 173, 222);">System.ComponentModel.PropertyDescriptorCollection</span>** 을 요청합니다.<br/>
따라서 뷰 포트에 표시될 바인딩 속성이 있다면 WPF 데이터 바인딩 엔진은 **<span style="color: rgb(107, 173, 222);">System.ComponentModel.ICustomTypeDescriptor.GetProperties()</span>** 메서드를 호출하고 해당 메서드에서 
바인딩 되는 속성들의 특성을 반환해 주면 동적 바인딩처리가 가능합니다.<br/>
이후엔 **<span style="color: rgb(107, 173, 222);">System.ComponentModel.PropertyDescriptor</span>** 클래스의 GetValue() 메서드로 노출될 바인딩 값을 가져와 렌더링 하게 되고 SetValue() 메서드로 값 변경처리를 하게 됩니다.<br/><br/>

그럼 예제코드를 통해 살펴 보겠습니다.<br/>
먼저 **<span style="color: rgb(107, 173, 222);">System.ComponentModel.PropertyDescriptor</span>** 클래스는 추상 클래스므로 해당 클래스를 상속받는 Custom클래스를 다음과 같이 작성해 봅니다.<br/>
**[CustomPropertyDescriptor.cs]**<br/>
```cs
using System;
using System.ComponentModel;

namespace ICustomTypeDescriptorEx;

public class CustomPropertyDescriptor<T> : PropertyDescriptor
{
    #region Member Fields
    private Type _propertyType;
    private Type _componentType;
    private T? _propertyValue;
    #endregion

    #region Constructor
    public CustomPropertyDescriptor(string propertyName, Type componentType)
        : base(propertyName, new Attribute[] { })
    {
        _propertyType = typeof(T?);
        _componentType = componentType;
    }
    #endregion

    #region PropertyDescriptor Implementation Overriden Methods
    public override bool CanResetValue(object component) { return true; }
    public override Type ComponentType
    {
        get
        {
            return _componentType;
        }
    }

    public override object? GetValue(object? component)
    {
        return _propertyValue;
    }

    public override bool IsReadOnly { get { return false; } }
    public override Type PropertyType { get { return _propertyType; } }
    public override void ResetValue(object component)
    {
        SetValue(component, default(T));
    }
    public override void SetValue(object? component, object? value)
    {
        if (value != null && value.GetType().IsAssignableFrom(_propertyType) == false)
        {
            throw new System.Exception("잘못된 타입입니다.");
        }

        _propertyValue = (T)value!;
    }

    public override bool ShouldSerializeValue(object component) { return true; }
    #endregion
}
```

그리고 **<span style="color: rgb(107, 173, 222);">System.ComponentModel.CustomTypeDescriptor</span>** 를 사용해서 동적 데이터 바인딩의 기본이 되는 ModelBase를 다음과 같이 작성합니다.<br/>
**[ModelBase.cs]**<br/>
```cs
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Linq;
using System.Runtime.CompilerServices;

namespace ICustomTypeDescriptorEx;

public class ModelBase : CustomTypeDescriptor, INotifyPropertyChanged
{
    #region Member Fields
    List<PropertyDescriptor> _properties = new List<PropertyDescriptor>();
    #endregion

    #region Public Methods
    /// <summary>
    /// 속성 설정
    /// </summary>
    /// <typeparam name="T">속성 타입</typeparam>
    /// <param name="propertyName">속성 이름</param>
    /// <param name="propertyValue">속성 값</param>
    public void SetPropertyValue<T>(string propertyName, T propertyValue)
    {
        var properties = this.GetProperties()
                                .Cast<PropertyDescriptor>()
                                .Where(prop => prop.Name.Equals(propertyName));

        if (properties == null || properties.Count() != 1)
        {
            throw new Exception($"{propertyName} 속성이 없습니다.");
        }

        var property = properties.First();
        property.SetValue(this, propertyValue);

        OnPropertyChanged(propertyName);
    }

    /// <summary>
    /// 속성 값 불러오기
    /// </summary>
    /// <typeparam name="T">속성 타입</typeparam>
    /// <param name="propertyName">속성 이름</param>
    /// <returns></returns>
    public T? GetPropertyValue<T>(string propertyName)
    {
        var properties = this.GetProperties()
                            .Cast<PropertyDescriptor>()
                            .Where(prop => prop.Name.Equals(propertyName));

        if (properties == null || properties.Count() != 1)
        {
            throw new Exception($"{propertyName} 속성이 없습니다.");
        }

        var property = properties.First();
        return (T?)property.GetValue(this);
    }

    /// <summary>
    /// 속성 추가
    /// </summary>
    /// <typeparam name="T">속성 타입</typeparam>
    /// <typeparam name="U">속성 Onwer (Model)</typeparam>
    /// <param name="propertyName"></param>
    public void AddProperty<T, U>(string propertyName)
        where U : ModelBase
    {
        var customProperty =
                new CustomPropertyDescriptor<T>(
                                        propertyName,
                                        typeof(U));

        _properties.Add(customProperty);
        customProperty.AddValueChanged(
                                    this,
                                    (o, e) => { OnPropertyChanged(propertyName); });
    }
    #endregion

    #region CustomTypeDescriptor Implementation Overriden Methods
    public override PropertyDescriptorCollection GetProperties()
    {
        var properties = base.GetProperties();
        return new PropertyDescriptorCollection(
                            properties.Cast<PropertyDescriptor>()
                                      .Concat(_properties).ToArray());
    }
    #endregion

    #region INotifyPropertyChange Implementation
    public event PropertyChangedEventHandler? PropertyChanged = delegate { };
    protected void OnPropertyChanged([CallerMemberName] string? propertyName = null)
    {
        PropertyChangedEventHandler? handler = this.PropertyChanged;

        if (handler != null)
        {
            handler(this, new PropertyChangedEventArgs(propertyName));
        }
    }
    #endregion INotifyPropertyChange Implementation
}
```

이제 위에서 만든 ModelBase클래스를 이용해서 동적 바인딩 처리가 가능합니다. 사용방법은 다음과 같습니다.<br/>
**[DataModel.cs]**<br/>
```cs
namespace ICustomTypeDescriptorEx;

public class DataModel : ModelBase
{
    //public string? UserID
    //{
    //    get;set;
    //}

    //public string? UserName
    //{
    //    get; set;
    //}

    //public bool UserChecked
    //{
    //    get; set;
    //}
}
```
**(데이터 바인딩 대상 클래스인 비어있는 DataModel 클래스)**<br/><br/>

**[MainViewModel.cs]**
```cs
using System;
using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Input;

namespace ICustomTypeDescriptorEx;

public class MainViewModel
{
    ObservableCollection<DataModel> _dataModelList = new ObservableCollection<DataModel>();

    public MainViewModel() {
        var data1 = this.InitDataModel();
        var data2 = this.InitDataModel();
        var data3 = this.InitDataModel();
        var data4 = this.InitDataModel();

        data1.SetPropertyValue<string>("UserID", Guid.NewGuid().ToString());
        data1.SetPropertyValue<string>("UserName", "arong");
        data1.SetPropertyValue<bool>("UserChecked", true);

        data2.SetPropertyValue<string>("UserID", Guid.NewGuid().ToString());
        data2.SetPropertyValue<string>("UserName", "ming");
        data2.SetPropertyValue<bool>("UserChecked", true);

        data3.SetPropertyValue<string>("UserID", Guid.NewGuid().ToString());
        data3.SetPropertyValue<string>("UserName", "test1");
        data3.SetPropertyValue<bool>("UserChecked", false);

        data4.SetPropertyValue<string>("UserID", Guid.NewGuid().ToString());
        data4.SetPropertyValue<string>("UserName", "test2");
        data4.SetPropertyValue<bool>("UserChecked", false);

        
        _dataModelList.Add(data1);
        _dataModelList.Add(data2);
        _dataModelList.Add(data3);
        _dataModelList.Add(data4);
    }

    public ObservableCollection<DataModel> DataModelList
    {
        get { return _dataModelList; }
    }

    private DataModel InitDataModel()
    {
        DataModel dataModel = new();
        dataModel.AddProperty<string, DataModel>("UserID");
        dataModel.AddProperty<string, DataModel>("UserName");
        dataModel.AddProperty<bool, DataModel>("UserChecked");
        return dataModel;
    }

    private RelayCommand<Object>? _updatePropertyCommand;
    public ICommand UpdatePropertyCommand
    {
        get
        {
            return _updatePropertyCommand ??
                (_updatePropertyCommand = new RelayCommand<Object>(this.UpdatePropertyExecute));
        }
    }

    private void UpdatePropertyExecute(object param)
    {
        DataModelList[1].SetPropertyValue<bool>("UserChecked", false);
    }
}
```

MainViewModel에서는 DataModel 인스턴스를 생성하고 동적으로 바인딩 대상의 속성 특성을 추가합니다. 그리고 그렇게 추가된 DataModel의 컬렉션을 리스트뷰에 바인딩 처리 합니다.<br/>
**[MainWindow.xaml]**
```xml
<Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="*"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>
        
        <ListView Grid.Row="0"
                  ItemsSource="{Binding DataModelList}">
            <ListView.View>
                <GridView>
                    <GridViewColumn Width="100"
                                    Header="UserID">
                        <GridViewColumn.CellTemplate>
                            <DataTemplate>
                                <TextBlock Text="{Binding UserID}"/>
                            </DataTemplate>
                        </GridViewColumn.CellTemplate>
                    </GridViewColumn>

                    <GridViewColumn Width="100"
                                    Header="UserName">
                        <GridViewColumn.CellTemplate>
                            <DataTemplate>
                                <TextBlock Text="{Binding UserName}"/>
                            </DataTemplate>
                        </GridViewColumn.CellTemplate>
                    </GridViewColumn>

                    <GridViewColumn Width="100"
                                    Header="UserChecked">
                        <GridViewColumn.CellTemplate>
                            <DataTemplate>
                                <CheckBox IsChecked="{Binding UserChecked}"/>
                            </DataTemplate>
                        </GridViewColumn.CellTemplate>
                    </GridViewColumn>
                </GridView>
            </ListView.View>
        </ListView>

        <StackPanel Grid.Row="1">
            <Button Content="Update property"
                    Height="30"
                    Command="{Binding UpdatePropertyCommand}"/>
        </StackPanel>
</Grid>
```

버튼을 통해 특정 속성에 대한 값 변경시 **<span style="color: rgb(107, 173, 222);">System.ComponentModel.PropertyDescriptor</span>** 클래스의 SetValue() 메서드로 값이 할당되고 GetValue() 메서드로 값 변경된 값을 불러오는걸 확인해 볼 수 있습니다.<br/>
마찬가지로 "UserChecked"속성 변경 또한 체크박스 변경시 반영이 잘되는걸 볼 수 있습니다.<br/><br/>


***

위 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
[Code_check - CustomTypeDescriptorEx](https://github.com/tyeom/code_check/tree/main/TestSample/csharp/CustomTypeDescriptorEx)



{% include content_adsense.html %}
