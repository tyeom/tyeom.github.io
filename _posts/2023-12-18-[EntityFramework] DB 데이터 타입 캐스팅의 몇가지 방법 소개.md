---
title: (EntityFramework) DB 데이터 타입 캐스팅의 몇가지 방법 소개
categories: EntityFramework
key: 20231218_01
comments: true
tags: EntityFramework EF ASP.NET MVC ASP.NET_MVC DB데이터타입 DB캐스팅 데이터타입변환
---

기존 DB 테이블 스키마 설계가 잘되어있는 DB에 Entity Framework(이하 EF) ORM을 사용하여 DB 데이터를 다룰 수 있다면 좋겠지만,
데이터 타입이 모두 문자열로 되어 있거나 또는 EF 사용시 데이터 비즈니스 로직 사용부분과 다른 타입인 경우 매번 데이터 조회시 해당 타입에 맞게
변환해서 사용해야 하는 경우가 있을 수 있습니다.<br/>
이러한 상황에서 EF ORM을 사용하는 DB Provider에 함수 매핑 지원이 안된다면 난감한 상황에 처할 수 있습니다.<br/>
이번 포스팅에서는 DB의 데이터 타입 변환 방법에 대해 살펴 봅니다.

<!--more-->

> 이 글에서 다루는 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
> [code_check](https://github.com/tyeom/code_check/tree/main/TestSample/aspnet_core/ef_sql_function_mapping)

문제의 상황
-

이미 구축된 서비스가 사용하는 DB에 다음과 같이 테이블 스키마 정의가 되어 있습니다.<br/>
**[테이블 : RealizedPl]**<br/>
<table>
        <tr>
                <th>열 이름</th>
                <th>데이터 형식</th>
        </tr>
        <tr>
                <td>No</td>
                <td>BIGINT</td>
        </tr>
        <tr>
                <td>CreateDate</td>
                <td>NVARCHAR(MAX)</td>
        </tr>
</table>

EF ORM을 도입하고 CreateDate 컬럼 기준으로 데이터를 조회 하고자 합니다.<br/>

Entity는 다음과 같이 정의 되어 있습니다.<br/><br/>

**[RealizedPlEntity.cs]**<br/>
```cs
public record RealizedPlEntity : IEntity
{
    [Key]
    public long No { get; set; }

    /// <summary>
    /// 문자 -> 날짜 변환 목표
    /// </summary>
    [Required]
    public string CreateDate { get; set; }
}
```

***

이렇게 Entity를 정의하고 데이터 조회를 위해 다음과 같이 시도해 볼 수 있겠지만..<br/><br/>

**[Controller.cs]**<br/>
```cs
public class PublicController : Controller
{
    private readonly IService _service = null!;

    public PublicController(IService service)
    {
        _service = service;
    }

    [HttpGet]
    public async Task<IActionResult> Get(DateTime createDate)
    {
        var allRealizedPl = await _service.Get(DateOnly.FromDateTime(createDate));
        return Ok(allRealizedPl);
    }
}
```

**[Service.cs]**<br/>
```cs
public class Service : IService
{
    private readonly Repository _repository;

    public async Task<IEnumerable<RealizedPlEntity>> Get(DateOnly createDate)
    {
        return await _repository.FindAll(p => DateOnly.ParseExact(p.CreateDate, "yyyy-MM-dd") >= createDate);
    }
}
```

아쉽게도 아래와 같은 오류가 발생 합니다.<br/><br/>
```
could not be translated. Additional information: Translation of method 'System.DateOnly.ParseExact' failed.
```
즉 위에서 데이터 변환을 위해 사용했던 **DateOnly.ParseExact() 메서드**는 해당 **DataBase Provier에서 제공하는 함수 매핑이 존재 하지 않기에** 위와 같은 오류가 발생되고 있습니다.<br/>
> **MSSQL의 EF Function Mappings 정보 참고**<br/>
> [Function Mappings of the Microsoft SQL Server Provider](https://learn.microsoft.com/en-us/ef/core/providers/sql-server/functions)

이런 상황을 해결 하기 위해 몇가지 방법을 살펴 보겠습니다.

첫번째 방법 : 암시적 데이터 변환
-

단순히 다음과 같이 Boxing처리 해서 MSSQL에서 제공하는 SQL 함수 CAST로 변환 되도록 처리 할 수 있습니다.<br/><br/>
**[Service.cs]**<br/>
```cs
public class Service : IService
{
    private readonly Repository _repository;

    public async Task<IEnumerable<RealizedPlEntity>> Get(DateOnly createDate)
    {
        return await _repository.FindAll(p => (DateOnly)(object)(p.CreateDate) >= createDate);
    }
}
```

암시적 데이터 변환 사용은 EF는 MSSQL DB 사용시 다음과 같은 쿼리를 생성하게 됩니다.<br/><br/>
**[SQL]**
```sql
CAST([r].[CreateDate] AS date) >=  @__createDate_0
```

하지만 이 방식은 몇가지 문제점이 존재 합니다.<br/>
1. 몇몇 특정 DataBase Provider에서는 지원하지 않을 수 있습니다.
2. 데이터 타입 변환 처리가 DataBase 내부의 메모리에서 처리 되어 단위 테스트시 정확한 결과값을 도출 하기 어렵습니다.
3. 다국어 처리에 있어서 특정 날짜 포맷 형태를 인식 할 수 없어 SQL 오류가 발생될 여지가 있습니다.


두번째 방법 : EF제공 Converter 사용
-

EF의 Fluent API를 사용해서 모델에 Converter를 사용 할 수 있습니다.<br/>
EF Converter는 **<span style="color: rgb(107, 173, 222);">Microsoft.EntityFrameworkCore.ModelBuilder.HasConversion() 메서드</span>** 로 적용 가능한데
DB에 데이터를 반영할때, 데이터를 비교할때, 데이터가 변경 되었을때 값 변환 처리를 제공 합니다.<br/>
우선 Converter를 적용하기 위해 기존 Entity모델에 문자 타입으로 되어 있는 날짜 데이터를 날짜 데이터로 변경 합니다.<br/><br/>

**[RealizedPlEntity.cs]**<br/>
```cs
public record RealizedPlEntity : IEntity
{
    [Key]
    public long No { get; set; }

    /// OnModelCreating.HasConversion() 사용
    [Required]
    public DateOnly CreateDate { get; set; }
}
```

그리고 Fluent API로 모델 설정을 다음과 같이 처리 합니다.<br/><br/>

**[DBContext.cs]**<br/>
```cs
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder
    .Entity<RealizedPlEntity>()
    .Property(c => c.CreateDate)
    .HasConversion(c => c.ToString("yyyyMMdd"),
                   c => DateOnly.ParseExact(c, "yyyyMMdd"));
}
```

**<span style="color: rgb(107, 173, 222);">Microsoft.EntityFrameworkCore.ModelBuilder.HasConversion() 메서드</span>**의 첫번째 표현식은
DB에 데이터 추가시 날짜 타입을 문자열로 변환해서 처리 하고, 두번째 표현식은 DB에서 데이터 조회시 기존 날짜 형태의 문자 데이터를 날짜 타입으로 변환해서 처리하도록 정의 한 것입니다.<br/>
이렇게 하면 데이터 조회시 EF LINQ에서는 날짜 타입을 다루듯 다음과 같이 간단하게 사용 가능 합니다.<br/><br/>

**[Service.cs]**<br/>
```cs
public class Service : IService
{
    private readonly Repository _repository;

    public async Task<IEnumerable<RealizedPlEntity>> Get(DateOnly createDate)
    {
        return await _revenueRepository.FindAll(p => p.CreateDate >= createDate);
    }
}
```

실제 위 표현식으로 만들어지는 MS SQL 은 다음과 같습니다.<br/><br/>

**[SQL]**
```sql
SELECT [r].[No], [r].[CreateDate], [r].[Name], [r].[RealizedPL]
      FROM [RealizedPlEntity] AS [r]
      WHERE [r].[CreateDate] >= @__createDate_0
```

보다시피 SQL에서는 데이터 변환을 위해 CAST를 사용하지 않고 순수하게 데이터 타입을 다루듯 쿼리가 생성 되어 SQL에 직접적인 영향을 주지 않습니다.<br/>
이런 경우 생성된 SQL이 실제 데이터 처럼 조회 되는 쿼리가 아니기 때문에 자칫 혼동을 일으킬 수 있습니다.<br/><br/>

또 다른 문제점은 첫번째 방법의 암시적 데이터 변환 방법의 단점중 날짜 포맷 처리가 고정 되어 다국어 설정에 따른 다른 형식의 날짜 포맷 고려가 되지 않는 부분 입니다.<br/>
날짜 데이터가 다음과 같이 일관 되어 있는 경우 의도한대로 정상 동작 합니다.<br/>
**[테이블 : RealizedPl]**<br/>
<table>
        <tr>
                <th>CreateDate</th>
        </tr>
        <tr>
                <td>20231218</td>
        </tr>
        <tr>
                <td>20231211</td>
        </tr>
        <tr>
                <td>20231210</td>
        </tr>
</table>

**[Curl]**
```
curl -X 'GET' \
  '?createDate=2023-12-11' \
  -H 'accept: */*'
```
**[Server response]**
```
[
  {
    "no": 1,
    "createDate": "2023-12-18"
  },
  {
    "no": 2,
    "createDate": "2023-12-11"
  }
]
```

그런데 날짜 포맷이 다른 데이터가 섞이는 경우<br/>
**[테이블 : RealizedPl]**<br/>
<table>
        <tr>
                <th>CreateDate</th>
        </tr>
        <tr>
                <td>2024-02-04 14:22:15</td>
        </tr>
</table>

**[Server response]**
```
System.FormatException: String '2024-02-04 14:22:15' was not recognized as a valid DateOnly.
```

위와 같이 System.FormatException 오류가 발생 됩니다.<br/>
이런 날짜 포맷을 상황에 맞게 적절히 처리하기 위해서 MSSQL을 사용하는 경우 CONVERT() 함수를 사용하도록 직접 사용자 정의 매핑을 사용 할 수 있습니다.


세번째 방법 : 사용자 정의 함수 매핑 사용
-

**<span style="color: rgb(107, 173, 222);">Microsoft.EntityFrameworkCore.ModelBuilder</span>** 클래스의 확장 **<span style="color: rgb(107, 173, 222);">Microsoft.EntityFrameworkCore.HasDbFunction()</span>** 메서드를 사용해서 직접 사용자 SQL 쿼리 생성 함수 매핑을 정의할 수 있습니다.<br/>
구현 방법은 다음과 같습니다.<br/>
먼저 **<span style="color: rgb(107, 173, 222);">Microsoft.EntityFrameworkCore.ModelBuilder</span>** 클래스의 확장 메서드를 만들어서 **<span style="color: rgb(107, 173, 222);">Microsoft.EntityFrameworkCore.HasDbFunction()</span>** 메서드 호출을 정의 합니다.<br/><br/>

**[Extensions/ModelBuilderExtensions.cs]**<br/>
```cs
public static class ModelBuilderExtensions
{
    public static DateTime? ToDateTime(this string strDate, int format) => throw new NotSupportedException("Use inside AddSqlDateOnlyConvertFunction");

    public static ModelBuilder AddSqlDateOnlyConvertFunction(this ModelBuilder modelBuilder)
    {
        modelBuilder.HasDbFunction(() => ToDateTime(default, default))
            .HasTranslation(args => new SqlFunctionExpression(
                    functionName: "CONVERT",
                    arguments: args.Prepend(new SqlFragmentExpression("DATETIME")),
                    nullable: false,
                    argumentsPropagateNullability: new[] { false, true, false },
                    type: typeof(DateTime),
                    typeMapping: null));

        return modelBuilder;
    }
}
```

이렇게 EF 에서 DB Provider중 MSSQL 인 경우 DateTime에 대한 CONVERT가 제공 되지 않았는데, 직접 ToDateTime() 메서드 사용으로 SQL 쿼리 생성시 CONVERT() 함수 사용으로 매핑 시킬 수 있습니다.<br/><br/>

참고로 **<span style="color: rgb(107, 173, 222);">Microsoft.EntityFrameworkCore.HasDbFunction()</span>** 메서드의 argumentsPropagateNullability 파라메터 값의 뜻은
사용자 정의 함수의 매개변수에 대한 null 전파 여부를 나타냅니다.<br/>
MSSQL의 CONVERT() 함수는 'target_type', 'expression', 'date_style smallint'의 총 3개의 파라메터 시그니처를 갖고 있습니다.<br/>
argumentsPropagateNullability가 true인 경우 null이 전파 된다는 것이고 false인 경우 null 전파가 되지 않는 다는 뜻 입니다.<br/>
true로 설정된 파라메터가 null로 전달되는 경우 해당 함수의 결과는 null로 반환 됩니다.<br/>
이렇게 구현한 메서드는 다음과 같이 처리 합니다.<br/><br/>

**[DBContext.cs]**<br/>
```cs
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // 사용자 정의 SQL함수 매핑 정의 사용 (HasTranslation)
    if (Database.IsSqlServer())
    {
        modelBuilder.AddSqlDateOnlyConvertFunction();
    }
}
```

그러면 이제 EF LINQ 사용시 ToDateTime() 메서드 사용은 SQL 생성시 CONVERT() 함수로 매핑되어 SQL 쿼리가 생성 됩니다.<br/>
실제 EF LINQ 에서는 다음과 같이 사용 합니다.<br/><br/>

**[Service.cs]**<br/>
```cs
public class Service : IService
{
    private readonly Repository _repository;

    public async Task<IEnumerable<RealizedPlEntity>> Get(DateOnly createDate)
    {
        return await _revenueRepository.FindAll(p => p.CreateDate.ToDateTime(20) >= createDate);
    }
}
```

생성된 SQL은 다음과 같습니다.<br/><br/>

**[SQL]**
```sql
SELECT [r].[No], [r].[CreateDate], [r].[Name], [r].[RealizedPL]
      FROM [RealizedPlEntity] AS [r]
      WHERE CONVERT(DATETIME, [r].[CreateDate], 20) >= @__createDate_0
```

이렇게 사용자 정의 함수 매핑 처리는 날짜 포맷 형식을 SQL 내에서 데이터 타입 변환으로 처리하기 때문에 상황에 맞게 명확한 처리 방법이 될 수 있습니다.


마무리
-

어쩔 수 없는 상황에서 데이터 변환이 이루어 져야 하는데 EF에서 DB Provider에 지원되지 않는 경우 처리 방법에 대해 살펴 보았습니다.<br/>
하지만 가장 좋은 방법은 할수만 있다면 데이터 성격에 맞게 테이블 스키마를 재정의 해서 DB 튜닝이 이루어 지는 것이 가장 좋은 방법 입니다.


***


위 코드는 다음 Repository에서 확인할 수 있습니다.<br/>
> [code_check](https://github.com/tyeom/code_check/tree/main/TestSample/aspnet_core/ef_sql_function_mapping)



{% include content_adsense.html %}
