---
title: (기타) GitHub Pages에 Blazor Webassembly 배포하기
categories: Etc
key: 20220928_01
comments: true
tags: GitHub GitHub_Actions Blazor Webassembly BlazorWebassembly 배포 deploy Actions
---

GitHub의 기능중 GitHub Pages로 무료 호스팅 기능을 제공합니다.<br/>
GitHub Pages에 호스팅을 위해 GitHub Actions를 사용하여 빌드 부터 배포까지 워크플로우를 생성해 자동화할 수 있는데 이번글에서 알아보도록 해보겠습니다.

<!--more-->

Blazor Webassembly 프로젝트 생성
-

우선 배포할 웹사이트를 만들기 위해 Blazor Webassembly 프로젝트를 생성해서 샘플 사이트를 만들어봅니다.<br/>
![image](https://user-images.githubusercontent.com/13028129/192689089-935194ba-2b55-4258-b0a6-424f17e50fdb.png)<br/><br/>

Blazor Webassembly 프로젝트가 생성되면 기본 샘플 템플릿으로 웹 어쎔블리 웹사이트가 생성됩니다. 빌드해서 정상 실행되는지 확인해 봅니다.

Repository 생성 및 Actions 워크플로우 설정
-

코드를 빌드하고 배포까지 자동화 처리를 위해 Actions에 워크플로우 설정이 필요합니다.<br/>
Actions 페이지로 이동해서 .NET 플랫폼 환경의 Action 환경설정을 추가 합니다.<br/>
![image](https://user-images.githubusercontent.com/13028129/192693290-1fc32324-ef7c-462c-ac35-61291f07e32b.png)<br/>
Configure를 눌러서 워크 플로우 설정 파일을 생성합니다.<br/>

**[dotnet.yml]**
```
name: .NET

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3
    - name: Setup .NET
      uses: actions/setup-dotnet@v2
      with:
        dotnet-version: 6.0.x
    - name: Restore dependencies
      run: dotnet restore
      working-directory: Blazor_wasm_Test/Blazor_wasm_Test
    - name: Build
      run: dotnet build --no-restore
      working-directory: Blazor_wasm_Test/Blazor_wasm_Test
    - name: Test
      run: dotnet test --no-build --verbosity normal
      working-directory: Blazor_wasm_Test/Blazor_wasm_Test
    - name: Publish with dotnet
      run: dotnet publish --configuration Release --output build
      working-directory: Blazor_wasm_Test/Blazor_wasm_Test
    - name: Deploy to Github Pages
      uses: JamesIves/github-pages-deploy-action@releases/v3
      with:
        ACCESS_TOKEN: {GitHub개인키}
        BASE_BRANCH: main # The branch the action should deploy from.
        BRANCH: master # The branch the action should deploy to.
        FOLDER: Blazor_wasm_Test/Blazor_wasm_Test/build/wwwroot # The folder the action should deploy.
        SINGLE_COMMIT: true
```
<br/>
설정시 확인해야 할 부분이 working-directory를 해당 Repository기준의 경로에 맞게 기입해야 합니다.<br/>
그리고 'Deploy to Github Pages' 단계에서 with의 ACCESS_TOKEN 부분에 github개인키를 입력하고 배포대상 FOLDER 경로 지정시 {Repository 기준경로}/build/wwwroot 로 되어야 합니다.<br/>
실제 빌드시 build폴더 하위로 빌드 결과 파일이 생성됩니다. 따라서 배포시에 필요한 파일들은 {Repository 기준경로}/build/wwwroot 폴더가 대상 입니다.<br/>
with의 BASE_BRANCH 부분은 배포할 작업이 되는 branch명이고 BRANCH는 배포 대상의 branch명 입니다. 여기서는 master로 설정해주고 이후에 master branch를 생성해 줍니다.<br/>
이렇게 해서 Actions의 워크플로우 설정파일을 생성 합니다.<br/><br/>

> **※ 참고로 ACCESS_TOKEN의 개인키는 GitHub계정 설정 -> Developer settings > Personal access tokens 에서 생성할 수 있습니다.**<br/>
![image](https://user-images.githubusercontent.com/13028129/192908293-7c9ee827-1b03-4f9a-a212-c605d043ee83.png)


다음으로는 GitHub Pages에 호스팅하기 위해 소스코드를 형상관리할 Repository를 생성하고 해당 코드를 Commit합니다.<br/>
아마 default branch가 **`main`** 으로 설정되어 있을텐데 GitHub Pages에 배포 대상으로 사용할 **`master`** branch를 추가로 생성합니다.<br/>
![image](https://user-images.githubusercontent.com/13028129/192690428-05930877-127d-408d-a372-3e70dcc0e8d8.png)<br/>
![image](https://user-images.githubusercontent.com/13028129/192690452-3e364dd9-f682-4c25-b2ee-015e562fe9ce.png)

Pages 설정
-

자동 빌드 및 배포 후 호스팅하기 위한 Pages를 설정해봅니다.<br/>
리파지토리의 Settings페이지에서 Pages부분에 Branch를 **`master`** 로 설정합니다. 이 설정만으로 해당 리파지토리의 Page가 만들어 집니다.

추가사항
-

GitHub의 Page는 Jekyll을 사용해서 호스팅 되는데 여기서는 Jekyll을 사용하지 않고 Blazor Webassembly를 빌드해서 배포된 파일을 사용할 것이기 때문에<br/>
**`main`** branch의 Blazor소스의 /wwwroot하위에 비어 있는 .nojekyll 파일을 추가해야 합니다. 그리고 404.html 파일도 같이 추가합니다.<br/>
![image](https://user-images.githubusercontent.com/13028129/192702210-41a08801-710a-4c0b-8ab4-ea199145c7d3.png)<br/>

**[404.html]**
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Single Page Apps for GitHub Pages</title>
    <script type="text/javascript">
        // Single Page Apps for GitHub Pages
        // https://github.com/rafrex/spa-github-pages
        // Copyright (c) 2016 Rafael Pedicini, licensed under the MIT License
        // ----------------------------------------------------------------------
        // This script takes the current url and converts the path and query
        // string into just a query string, and then redirects the browser
        // to the new url with only a query string and hash fragment,
        // e.g. http://www.foo.tld/one/two?a=b&c=d#qwe, becomes
        // http://www.foo.tld/?p=/one/two&q=a=b~and~c=d#qwe
        // Note: this 404.html file must be at least 512 bytes for it to work
        // with Internet Explorer (it is currently > 512 bytes)
        // If you're creating a Project Pages site and NOT using a custom domain,
        // then set segmentCount to 1 (enterprise users may need to set it to > 1).
        // This way the code will only replace the route part of the path, and not
        // the real directory in which the app resides, for example:
        // https://username.github.io/repo-name/one/two?a=b&c=d#qwe becomes
        // https://username.github.io/repo-name/?p=/one/two&q=a=b~and~c=d#qwe
        // Otherwise, leave segmentCount as 0.
        var segmentCount = 0;
        var l = window.location;
        l.replace(
            l.protocol + '//' + l.hostname + (l.port ? ':' + l.port : '') +
            l.pathname.split('/').slice(0, 1 + segmentCount).join('/') + '/?p=/' +
            l.pathname.slice(1).split('/').slice(segmentCount).join('/').replace(/&/g, '~and~') +
            (l.search ? '&q=' + l.search.slice(1).replace(/&/g, '~and~') : '') +
            l.hash
        );
    </script>
</head>
<body>
</body>
</html>
```

마지막으로 index.html파일에 다음 JavaScript 코드를 추가하고 head태그의 base 태그 부분을 리파지토리 이름으로 변경해줍니다.<br/>
```js
<!-- Start Single Page Apps for GitHub Pages -->
    <script type="text/javascript">
        // Single Page Apps for GitHub Pages
        // https://github.com/rafrex/spa-github-pages
        // Copyright (c) 2016 Rafael Pedicini, licensed under the MIT License
        // ----------------------------------------------------------------------
        // This script checks to see if a redirect is present in the query string
        // and converts it back into the correct url and adds it to the
        // browser's history using window.history.replaceState(...),
        // which won't cause the browser to attempt to load the new url.
        // When the single page app is loaded further down in this file,
        // the correct url will be waiting in the browser's history for
        // the single page app to route accordingly.
        (function (l) {
            if (l.search) {
                var q = {};
                l.search.slice(1).split('&').forEach(function (v) {
                    var a = v.split('=');
                    q[a[0]] = a.slice(1).join('=').replace(/~and~/g, '&');
                });
                if (q.p !== undefined) {
                    window.history.replaceState(null, null,
                        l.pathname.slice(0, -1) + (q.p || '') +
                        (q.q ? ('?' + q.q) : '') +
                        l.hash
                    );
                }
            }
        }(window.location))
    </script>
    <!-- End Single Page Apps for GitHub Pages -->
```

```html
<head>
  ...생략...
  <base href="/Blazor_Test/" />
  ...생략...
</head>
```

최종 index.html은 다음과 같습니다.<br/>
**[index.html]**
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
    <title>Blazor_wasm_Test</title>
    <base href="/Blazor_Test/" />
    <link href="css/bootstrap/bootstrap.min.css" rel="stylesheet" />
    <link href="css/app.css" rel="stylesheet" />
    <link href="Blazor_wasm_Test.styles.css" rel="stylesheet" />
</head>

<body>
    <div id="app">Loading...</div>
    
    <!-- Start Single Page Apps for GitHub Pages -->
    <script type="text/javascript">
        // Single Page Apps for GitHub Pages
        // https://github.com/rafrex/spa-github-pages
        // Copyright (c) 2016 Rafael Pedicini, licensed under the MIT License
        // ----------------------------------------------------------------------
        // This script checks to see if a redirect is present in the query string
        // and converts it back into the correct url and adds it to the
        // browser's history using window.history.replaceState(...),
        // which won't cause the browser to attempt to load the new url.
        // When the single page app is loaded further down in this file,
        // the correct url will be waiting in the browser's history for
        // the single page app to route accordingly.
        (function (l) {
            if (l.search) {
                var q = {};
                l.search.slice(1).split('&').forEach(function (v) {
                    var a = v.split('=');
                    q[a[0]] = a.slice(1).join('=').replace(/~and~/g, '&');
                });
                if (q.p !== undefined) {
                    window.history.replaceState(null, null,
                        l.pathname.slice(0, -1) + (q.p || '') +
                        (q.q ? ('?' + q.q) : '') +
                        l.hash
                    );
                }
            }
        }(window.location))
    </script>
    <!-- End Single Page Apps for GitHub Pages -->

    <div id="blazor-error-ui">
        An unhandled error has occurred.
        <a href="" class="reload">Reload</a>
        <a class="dismiss">🗙</a>
    </div>
    <script src="_framework/blazor.webassembly.js"></script>
</body>

</html>
```

이후 Actions 부분을 확인해 보면 자동 빌드 되고 **`master`** branch로 배포되는것을 확인할 수 있습니다.<br/>
![image](https://user-images.githubusercontent.com/13028129/192703882-1752225e-4028-48d9-a5b1-5f0c2ccd58db.png)<br/>
최종 URL은 https://{github ID}.github.io/Repository이름 입니다.<br/>
[https://blog.arong.info/Blazor_Test/](https://blog.arong.info/Blazor_Test/)

{% include content_adsense.html %}
