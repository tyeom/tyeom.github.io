---
title: (WPF) 두개의 DataGrid Cell 포커스 동기화 처리
categories: WPF
key: 20220615_01
comments: true
tags: WPF MVVM DataGrid ColumnCell CellFocus SelectCell Cell선택동기화 Behavior
---

WPF DataGrid Cell 선택 관련 질문에 대한 답변 포스트 입니다.

<!--more-->

WPF DataGrid 관련 질문이 있었는데 다음과 같은 내용 입니다. 요약하면

기본 DataGrid를 사용중이고 서로 다른 데이터 모델이 바인딩 되어 있는 두개의 DataGrid가 있는데<br/>
사용자가 선택한 Cell 위치에 대해 서로 똑같이 선택되도록 동기화 하려면 어떻게 해야 하나요?<br/>
입니다.

질문 분석 및 분해
-
우선 질문 내용을 분해해서 분석해 보면<br/>
- a. DataGrid에서 Row단위가 아닌 Column Cell 단위로 선택되도록 하는 방법
- b. DataGrid에서 Cell이 선택 되었을때 알 수 있는 방법 (이벤트 발생 또는 속성)
- c. DataGrid Cell Select() 또는 Focus() 하는 방법

크게 이렇게 질문을 분해해 볼 수 있습니다. 위 질문 분해에 대한 답을 찾으면 그때 부터는 어떻게 조립해서 깔끔하게 사용하게 할 수 있을지 고민 인거죠

우선 위에 나열한 3가지 부분은 MS DataGrid MS Doc 사이트와 구글검색을 통해 바로 해답을 찾을 수 있습니다.

질문 분해의 답
-

[SelectionUnit](https://docs.microsoft.com/ko-kr/dotnet/api/system.windows.controls.datagrid.selectionunit?view=windowsdesktop-6.0) 속성으로
Cell 단위 선택 설정이 가능하고,<br/>
[SelectedCellsChanged](https://docs.microsoft.com/ko-kr/dotnet/api/system.windows.controls.datagrid.selectedcellschanged?view=windowsdesktop-6.0) 이벤트로
Cell 선택 여부 확인이 가능하고,<br/>
[VisualTreeHelper](https://docs.microsoft.com/ko-kr/dotnet/api/system.windows.media.visualtreehelper?view=windowsdesktop-6.0)클래스를 사용해서 특정 DataGridCell을 찾아 Focus() 처리가 가능합니다.

그럼 방법은 찾았으니 어떻게 구현하면 좋을지 고민 시작 입니다.<br/>

추가 고민
-

단순히 위 방법대로 직접 해당 뷰에서 이벤트 핸들러를 추가해서 직접 처리해도 되겠지만 해당 처리가 한부분만 있는것이 아니라면<br/>
중복 코드가 발생 됩니다. 따라서 공통으로 처리할 수 있도록 하는것이 좋을 것 같습니다. 추가로 MVVM패턴에 맞게 바인딩 처리로 한다면 더욱 좋아보입니다.<br/>
그렇게 하려면 일단 당장 떠오르는 방법은 Behavior를 만들어서 바인딩으로 처리 하는 방법이 생각나 다음처럼 처리해 보았습니다.

질문 해결
-

DataGrid의 Cell 선택처리 및 SelectedCellsChanged이벤트를 활용해 Cell 변경시 Command로 제공받을 수 있는 Behavior를 만들어서 처리 해보았습니다.<br/>
Command와 같이 넘어오는 선택된 Cell Index정보는 간편하게 String으로 콤마(,) 구분의 {rowIndex},{columnIndex} 형태로 처리 했습니다.

**[DataGridCellBehavior.cs]**<br/>
```cs
using System;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Controls.Primitives;
using System.Windows.Input;
using System.Windows.Media;

internal class DataGridCellBehavior : DependencyObject
{
        public static readonly DependencyProperty IsCellFocusProperty =
            DependencyProperty.RegisterAttached("IsCellFocus",
                typeof(bool),
                typeof(DataGridCellBehavior),
                new UIPropertyMetadata(false, (d, e) =>
                {
                    DataGrid dataGrid = d as DataGrid;
                    if (dataGrid == null)
                    {
                        throw new ArgumentException("This property may only be used on DataGrid");
                    }

                    if (((bool)e.NewValue) is true)
                    {
                        dataGrid.SelectedCellsChanged += DataGrid_SelectedCellsChanged;
                    }
                    else
                    {
                        dataGrid.SelectedCellsChanged -= DataGrid_SelectedCellsChanged;
                    }
                }));

        private static void DataGrid_SelectedCellsChanged(object sender, SelectedCellsChangedEventArgs e)
        {
            var dataGrid = sender as DataGrid;
            if (dataGrid != null && e.AddedCells != null && e.AddedCells.Count > 0)
            {
                var cell = e.AddedCells[0];
                if (!cell.IsValid)
                    return;

                var generator = dataGrid.ItemContainerGenerator;
                int columnIndex = cell.Column.DisplayIndex;
                int rowIndex = generator.IndexFromContainer(generator.ContainerFromItem(cell.Item));

                ICommand selectedCellsChangedCommand = GetSelectedCellsChangedCommand(dataGrid);
                selectedCellsChangedCommand.Execute($"{rowIndex},{columnIndex}");
            }
        }

        public static bool GetIsCellFocus(DependencyObject obj)
        {
            return (bool)obj.GetValue(IsCellFocusProperty);
        }

        public static void SetIsCellFocus(DependencyObject obj, bool value)
        {
            obj.SetValue(IsCellFocusProperty, value);
        }

        public static ICommand GetSelectedCellsChangedCommand(DependencyObject obj)
        {
            return (ICommand)obj.GetValue(SelectedCellsChangedCommandProperty);
        }

        public static void SetSelectedCellsChangedCommand(DependencyObject obj, ICommand value)
        {
            obj.SetValue(SelectedCellsChangedCommandProperty, value);
        }

        public static readonly DependencyProperty SelectedCellsChangedCommandProperty =
            DependencyProperty.RegisterAttached(
                "SelectedCellsChangedCommand",
                typeof(ICommand),
                typeof(DataGridCellBehavior),
                new UIPropertyMetadata(null)
            );


        public static readonly DependencyProperty CellFocusProperty =
            DependencyProperty.RegisterAttached("CellFocus",
                typeof(string),
                typeof(DataGridCellBehavior),
                new UIPropertyMetadata(null, OnCellFocusPropertyChanged));

        public static string GetCellFocus(DependencyObject obj)
        {
            return (string)obj.GetValue(CellFocusProperty);
        }

        public static void SetCellFocus(DependencyObject obj, string value)
        {
            obj.SetValue(CellFocusProperty, value);
        }

        private static void OnCellFocusPropertyChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
        {
            if(e.NewValue == null) return;

            DataGrid dataGrid = (DataGrid)d;
            if (dataGrid.SelectionUnit != DataGridSelectionUnit.Cell)
                throw new ArgumentException("데이터그리드 SelectionUnit 속성이 Cell 모드가 아닙니다!");

            var row_cell_token = e.NewValue.ToString().Split(new string[] { "," }, StringSplitOptions.RemoveEmptyEntries);
            if(row_cell_token.Length != 2)
                throw new ArgumentException("데이터그리드 row, cell index 문자열 정보가 잘못 되었습니다!");

            int rowIndex = int.Parse(row_cell_token[0]);
            int columnIndex = int.Parse(row_cell_token[1]);

            if (rowIndex < 0 || rowIndex > (dataGrid.Items.Count - 1))
                throw new ArgumentException("유효하지 않은 Row Index!");

            if (columnIndex < 0 || columnIndex > (dataGrid.Columns.Count - 1))
                throw new ArgumentException("유효하지 않은 Column Index!");

            dataGrid.SelectedCells.Clear();

            object item = dataGrid.Items[rowIndex];
            DataGridRow row = dataGrid.ItemContainerGenerator.ContainerFromIndex(rowIndex) as DataGridRow;
            if (row == null)
            {
                dataGrid.ScrollIntoView(item);
                row = dataGrid.ItemContainerGenerator.ContainerFromIndex(rowIndex) as DataGridRow;
            }
            if (row != null)
            {
                DataGridCell cell = GetCell(dataGrid, row, columnIndex);
                if (cell != null)
                {
                    DataGridCellInfo dataGridCellInfo = new DataGridCellInfo(cell);
                    dataGrid.SelectedCells.Add(dataGridCellInfo);
                    cell.Focus();
                }
            }
        }

        public static DataGridCell GetCell(DataGrid dataGrid, DataGridRow rowContainer, int column)
        {
            if (rowContainer != null)
            {
                DataGridCellsPresenter presenter = FindVisualChild<DataGridCellsPresenter>(rowContainer);
                if (presenter == null)
                {
                    // CellsPresenter가 null인 경우 (아직 렌더링 전으로 추측?)
                    // DataGridRow의 ApplyTemplate()를 호출해줌으로써 비주얼트리가 적용되도록 처리
                    rowContainer.ApplyTemplate();
                    presenter = FindVisualChild<DataGridCellsPresenter>(rowContainer);
                }
                if (presenter != null)
                {
                    DataGridCell cell = presenter.ItemContainerGenerator.ContainerFromIndex(column) as DataGridCell;
                    if (cell == null)
                    {
                        // 해당 Cell이 데이터그리드 스크롤영역 밖으로 보이지 않은 경우
                        // 해당 Cell로 스크롤이동
                        dataGrid.ScrollIntoView(rowContainer, dataGrid.Columns[column]);
                        cell = presenter.ItemContainerGenerator.ContainerFromIndex(column) as DataGridCell;
                    }
                    return cell;
                }
            }
            return null;
        }

        public static T FindVisualChild<T>(DependencyObject obj) where T : DependencyObject
        {
            for (int i = 0; i < VisualTreeHelper.GetChildrenCount(obj); i++)
            {
                DependencyObject child = VisualTreeHelper.GetChild(obj, i);
                if (child != null && child is T)
                    return (T)child;
                else
                {
                    T childOfChild = FindVisualChild<T>(child);
                    if (childOfChild != null)
                        return childOfChild;
                }
            }
            return null;
        }
}
```

**[MainWindow.xaml]**<br/>
```xml
<Grid>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="*"/>
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>

        <DataGrid Grid.Column="0"
                  SelectionUnit="Cell"
                  local:DataGridCellBehavior.IsCellFocus="True"
                  local:DataGridCellBehavior.SelectedCellsChangedCommand="{Binding SelectedCellsChangedCommand}"
                  local:DataGridCellBehavior.CellFocus="{Binding Row_column, Mode=TwoWay}"
                  ItemsSource="{Binding AList}"/>

        <DataGrid Grid.Column="1"
                  SelectionUnit="Cell"
                  ItemsSource="{Binding BList}"
                  local:DataGridCellBehavior.IsCellFocus="True"
                  local:DataGridCellBehavior.SelectedCellsChangedCommand="{Binding SelectedCellsChangedCommand}"
                  local:DataGridCellBehavior.CellFocus="{Binding Row_column, Mode=TwoWay}"/>
</Grid>
```

**[Test Model]**
```cs
public class AModel
{
        public string Name
        {
            get;set;
        }

        public int Age
        {
            get; set;
        }
    }

    public class BModel
    {
        public string Status
        {
            get; set;
        }

        public bool IsCheck
        {
            get; set;
        }
}
```

**[MainViewModel.cs]**<br/>
```cs
public class MainViewModel : INotifyPropertyChanged
{
        private string _row_column = null;

        public MainViewModel()
        {
            List<AModel> aList = new List<AModel>();
            aList.Add(new AModel() { Name = "test1", Age = 11 });
            aList.Add(new AModel() { Name = "test2", Age = 22 });
            aList.Add(new AModel() { Name = "test3", Age = 33 });

            List<BModel> bList = new List<BModel>();
            bList.Add(new BModel() { Status = "Nomal", IsCheck = true });
            bList.Add(new BModel() { Status = "Hard", IsCheck = false });
            bList.Add(new BModel() { Status = "Mid", IsCheck = true });

            AList = aList;
            BList = bList;
        }

        public string Row_column
        {
            get => _row_column;
            set
            {
                _row_column = value;
                OnPropertyChanged();
            }

        }

        public List<AModel> AList
        {
            get;set;
        }

        public List<BModel> BList
        {
            get; set;
        }

        private RelayCommand<string> _selectedCellsChangedCommand;
        public ICommand SelectedCellsChangedCommand
        {
            get
            {
                return _selectedCellsChangedCommand ??
                    (_selectedCellsChangedCommand = new RelayCommand<string>(
                        param => this.ExecuteSelectedCellsChanged(param)));
            }
        }

        private void ExecuteSelectedCellsChanged(string param)
        {
            Row_column = param;
        }
        
        #region INotifyPropertyChange Implementation
        public event PropertyChangedEventHandler PropertyChanged = delegate { };
        protected void OnPropertyChanged([CallerMemberName] string propertyName = null)
        {
            PropertyChangedEventHandler handler = this.PropertyChanged;

            if (handler != null)
            {
                handler(this, new PropertyChangedEventArgs(propertyName));
            }
        }
        #endregion INotifyPropertyChange Implementation
}

internal class RelayCommand<T> : ICommand
{
        private readonly Action<T> _execute;

        public RelayCommand(Action<T> execute)
        {
            _execute = execute;
        }

        public bool CanExecute(object parameter)
        {
            return true;
        }

        public void Execute(object parameter)
        {
            T param = (T)parameter;
            _execute(param);
        }

        public event EventHandler CanExecuteChanged
        {
            add
            {
                CommandManager.RequerySuggested += value;
            }

            remove
            {
                CommandManager.RequerySuggested -= value;
            }
        }
}
```

![DataGridCellBehavior](https://user-images.githubusercontent.com/13028129/173708039-9c994a94-3b95-4d0e-aa71-2afcecf6ff94.gif)

전체 코드는 DataGridCellBehavior Repository에 있습니다.<br/>
[DataGridCellBehavior](https://github.com/tyeom/code_check/tree/main/QnA/WPF/DataGridCellBehavior)

{% include content_adsense.html %}
