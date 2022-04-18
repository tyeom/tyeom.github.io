---
title: (JavaScript)  editorjs 적용하기 with ASP.NET Core
categories: JavaScript
key: 20220418_01
comments: true
tags: editorjs, 웹에디터, javascript_editor
---

개인 사이트에 자바스크립트 웹에디터 라이브러리를 현재는 [summernote](https://summernote.org/) 를 적용해서 사용하고 있는데 추가적으로 다른 에디터를 적용해볼까
생각중에 [editorjs](https://editorjs.io/) 를 알게 되어 한번 적용해 보기로 했습니다.

<!--more-->

[editorjs](https://editorjs.io/) 에디터는 Notion(노션)에서의 노트 작성과 비슷한 UX환경을 제공합니다.<br/>
block단위로 단순 텍스트, 이미지, 테이블(표) 등을 삽입할 수 있고 위치를 이동할 수 있는 형태로 에디트 할 수 있습니다.

그럼 하나씩 editorjs 에디터를 적용해 보도록 하겠습니다.

editorjs 에디터를 적용해보려는 환경은 ASP.NET Core 3.1 환경인 웹사이트 입니다.<br/>
JavaScript 기반의 에디터이기 때문에 환경은 딱히 중요하진 않습니다.

JavaScript 기반의 editorjs의 샘플과 자세한 사용법 및 npm설치 방법, CDN 사용 설명은 git 페이지[https://github.com/codex-team/editor.js](https://github.com/codex-team/editor.js)에 상세하게 설명 되어 있습니다.<br/>

이번 포스팅의 editorjs 에디터는 git 페이지 데모 예제 기반으로 하나씩 적용해 보겠습니다.

뷰 에디터 적용 (Add)
-

css 파일 https://github.com/codex-team/editor.js/blob/next/example/assets/demo.css
에디터의 결과 미리보기 출력용도인 js파일 https://github.com/codex-team/editor.js/blob/next/example/assets/json-preview.js

위 두개의 파일을 적절한 위치에 다운로드 받습니다.

css파일과 js파일을 다운로드 받고 게시물을 작성할 수 있는 뷰 페이지를 생성해서 해당 페이지에 에디터를 적용합니다.

우선 다운로드 받은 css와 js 파일을 뷰 페이지에 참조 시켜 줍니다.<br/>
**[Shared/_Layout.cshtml]**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <link rel="stylesheet" href="~/css/editorjs.css">  <- 추가
    <script src="~/js/json-preview.js"></script>       <- 추가

...이하 생략...
```

**[Add.cshtml]**
```html
@model CoreNote.Models.Archive
@{
    ViewData["Title"] = "Archive Write";
}
<form class="table-hover" id="form" method="post" asp-controller="Archive" asp-action="Add">
  <div class="form-group">
    <input type="text" class="form-control" asp-for="Title" placeholder="제목 입력" />
    <span class="text-danger" asp-validation-for="Title"></span>
  </div>
  <input type="hidden" id="Contents" asp-for="Contents" />
</form>

<div class="ce-example__content _ce-example__content--small">
  <div id="editorjs"></div>
</div>
<div class="ce-example__output">
  <pre class="ce-example__output-content" id="output" asp-for="Contents"></pre>
</div>
<div class="form-group">
  <button class="btn btn-primary" id="saveButton">저장</button>
</div>        
```

제목과 에디터를 사용해서 본문을 작성할 수 있는 뷰 페이지 입니다. hidden 타입의 Contents 필드는 에디터로 작성된 결과 blocks json을 해당 필드에 설정해서 form data로 요청하는 용도 입니다.

그리고 saveButton 버튼을 클릭했을때 자바스크립트 함수를 통해 에디터 결과 데이터의 json을 output pre태그에 출력하고 form action url로 요청합니다.

editorjs 에디터는 사용할 기능을 자바스크립트 모듈화로 제공하게 되어 있습니다. 그렇기 때문에 에디터에서 사용할 기능은 각각 npm으로 다운받아 적용하거나 CDN url로 참조하여 설정해 줄 수 있습니다.<br/>
참조 git 페이지 : [https://github.com/orgs/editor-js/repositories?type=all](https://github.com/orgs/editor-js/repositories?type=all)

다음과 같이 CDN을 사용하여 참조 시켜 줍니다.<br/>
그리고 에디터 자체 모듈도 CDN을 사용하여 같이 참조 시켜 줍니다.<br/>
**[Add.cshtml]**
```js
<script src="https://cdn.jsdelivr.net/npm/@@editorjs/header@@latest"></script><!-- Header -->
  <script src="https://cdn.jsdelivr.net/npm/@@editorjs/simple-image@@latest"></script><!-- Image -->
  <script src="https://cdn.jsdelivr.net/npm/@@editorjs/image@2.3.0"></script><!-- Image Uploader -->
  <script src="https://cdn.jsdelivr.net/npm/@@editorjs/attaches@@latest"></script><!-- attaches -->
  <script src="https://cdn.jsdelivr.net/npm/@@editorjs/delimiter@@latest"></script><!-- Delimiter -->
  <script src="https://cdn.jsdelivr.net/npm/@@editorjs/list@@latest"></script><!-- List -->
  <script src="https://cdn.jsdelivr.net/npm/@@editorjs/checklist@@latest"></script><!-- Checklist -->
  <script src="https://cdn.jsdelivr.net/npm/@@editorjs/quote@@latest"></script><!-- Quote -->
  <script src="https://cdn.jsdelivr.net/npm/@@editorjs/code@@latest"></script><!-- Code -->
  <script src="https://cdn.jsdelivr.net/npm/@@editorjs/embed@@latest"></script><!-- Embed -->
  <script src="https://cdn.jsdelivr.net/npm/@@editorjs/table@@latest"></script><!-- Table -->
  <script src="https://cdn.jsdelivr.net/npm/@@editorjs/link@@latest"></script><!-- Link -->
  <script src="https://cdn.jsdelivr.net/npm/@@editorjs/warning@@latest"></script><!-- Warning -->
  <script src="https://cdn.jsdelivr.net/npm/@@editorjs/marker@@latest"></script><!-- Marker -->
  <script src="https://cdn.jsdelivr.net/npm/@@editorjs/inline-code@@latest"></script><!-- Inline Code -->
  
  <!-- Load Editor.js's Core -->
  <script src="https://cdn.jsdelivr.net/npm/@@editorjs/editorjs@@latest"></script>
```

위 처럼 에디터에서 사용할 기능 모듈을 참조 하고 에디터 라이브러리도 참조 하면 자바스크립트 EditorJS 모듈을 사용할 수 있습니다.

본격 적인 EditorJS모듈 설정 부분인 자바스크립트 코드 입니다. (Add뷰에 계속 이어서 작업)

**[Add.cshtml]**
```js
<script>
    var editor = new EditorJS({
      readOnly: false,
      holder: 'editorjs',
      
      /**
       * Tools list
       */
      tools: {
        header: {
          class: Header,
          inlineToolbar: ['marker', 'link'],
          config: {
            placeholder: 'Header'
          },
          shortcut: 'CMD+SHIFT+H'
        },
        
        image: {
            class: ImageTool,
            config: {
                uploader: {
                    uploadByFile(file) {
                        return uploadImage(file).then((resultUrl) => {
                            return {
                                success: 1,
                                file: {
                                    url: resultUrl
                                }
                            };
                        });
                    }
                },
            }
        },

        attaches: {
            class: AttachesTool,
            config: {
                uploader: {
                    uploadByFile(file) {
                        // your own uploading logic here
                        return uploadFile(file).then((resultData) => {
                            return {
                                success: 1,
                                file: {
                                    url: resultData.url,
                                    size: resultData.size,
                                    name: resultData.title,
                                    title: resultData.title,
                                    // any data you want 
                                    // for example: name, size, title
                                }
                            };
                        });
                    },
                }
            }
        },

        list: {
          class: List,
          inlineToolbar: true,
          shortcut: 'CMD+SHIFT+L'
        },

        checklist: {
          class: Checklist,
          inlineToolbar: true,
        },

        quote: {
          class: Quote,
          inlineToolbar: true,
          config: {
            quotePlaceholder: 'Enter a quote',
            captionPlaceholder: 'Quote\'s author',
          },
          shortcut: 'CMD+SHIFT+O'
        },

        warning: Warning,

        marker: {
          class:  Marker,
          shortcut: 'CMD+SHIFT+M'
        },

        code: {
          class:  CodeTool,
          shortcut: 'CMD+SHIFT+C'
        },

        delimiter: Delimiter,

        inlineCode: {
          class: InlineCode,
          shortcut: 'CMD+SHIFT+C'
        },

        linkTool: LinkTool,

        embed: Embed,

        table: {
          class: Table,
          inlineToolbar: true,
          shortcut: 'CMD+ALT+T'
        },

      },

      data: {
        blocks: [
          {
            type : 'paragraph',
            data : {
              text : '내용을 입력하세요.'
            }
          },
        ]
      },
      onReady: function(){},
      onChange: function(api, event) {
        console.log('something changed', event);
      }
    });

    const saveButton = document.getElementById('saveButton');

    const toggleReadOnlyButton = document.getElementById('toggleReadOnlyButton');
    const readOnlyIndicator = document.getElementById('readonly-state');
    
    saveButton.addEventListener('click', function () {
      editor.save()
        .then((savedData) => {
          cPreview.show(savedData, document.getElementById("output"));
          document.getElementById('Contents').value = JSON.stringify( savedData, null, 4 );
          document.getElementById('form').submit();
        })
        .catch((error) => {
          console.error('Saving error', error);
        });
    });

    function uploadImage(file) {
        let form_data = new FormData();
        form_data.append('file', file);

        return new Promise((resolve, reject) => {
            $.ajax({
                data: form_data,
                type: "POST",
                url: '/api/imageUpload',
                cache: false,
                contentType: false,
                enctype: 'multipart/form-data',
                processData: false,
                success: function (url) {
                    resolve(url)
                }
            });
        })
    }

    function uploadFile(file) {
        let form_data = new FormData();
        form_data.append('file', file);

        return new Promise((resolve, reject) => {
            $.ajax({
                data: form_data,
                type: "POST",
                url: '/api/fileUploadForEditorJs',
                cache: false,
                contentType: false,
                enctype: 'multipart/form-data',
                processData: false,
                success: function (resultData) {
                    resolve(resultData)
                }
            });
        })
    }
</script>
```

**ImageTool**과 **AttachesTool**은 Promise를 반환하면 비동기로 BackEnd EndPoint url로 호출해서 반환된 데이터를 사용할 수 있도록 제공 하고 있습니다.<br/>
**uploadImage(file)** 함수와 **uploadFile(file)** 함수를 정의해서 각각 해당 함수를 사용할 수 있도록 처리해 주었습니다.<br/>
두 기능 모두 업로드 처리 성공시 success 속성에 1로 반환을 시켜주어야 합니다. (실패시는 0 반환)<br/>
```js
function uploadImage(file) {
        let form_data = new FormData();
        form_data.append('file', file);

        return new Promise((resolve, reject) => {
            $.ajax({
                data: form_data,
                type: "POST",
                url: '/api/imageUpload',
                cache: false,
                contentType: false,
                enctype: 'multipart/form-data',
                processData: false,
                success: function (url) {
                    resolve(url)
                }
            });
        })
}

function uploadFile(file) {
        let form_data = new FormData();
        form_data.append('file', file);

        return new Promise((resolve, reject) => {
            $.ajax({
                data: form_data,
                type: "POST",
                url: '/api/fileUploadForEditorJs',
                cache: false,
                contentType: false,
                enctype: 'multipart/form-data',
                processData: false,
                success: function (resultData) {
                    resolve(resultData)
                }
            });
        })
}
```

그리고 saveButton 버튼 클릭시 json-preview.js 에 정의 되어 있는 cPreview를 사용해서 에디터 결과 데이터인 json 문자열을 output pre태그에 출력하고,<br/>
Contents 필드 값에도 json 문자를 적용하고 바로 form submit 되도록 처리 했습니다.<br/>
```js
saveButton.addEventListener('click', function () {
      editor.save()
        .then((savedData) => {
          cPreview.show(savedData, document.getElementById("output"));
          document.getElementById('Contents').value = JSON.stringify( savedData, null, 4 );
          document.getElementById('form').submit();
        })
        .catch((error) => {
          console.error('Saving error', error);
        });
});
```

이렇게 editorjs 에디터를 이용해 작성된 내용을 서버로 전송해서 데이터를 처리할 수 있습니다.<br/>

> ※ 기존 데이터를 다시 에디터 상으로 Load하는 방법은 EditorJS 모듈 초기화시에<br/>
> data 속성의 blocks 부분에 저장된 json데이터를 적용 시켜주면 됩니다.
> 아래 코드 참고
```cs
[Required(ErrorMessage = "내용을 입력해 주세요!")]
public string Contents { get; set; }

[NotMapped]
public string ContentBlocks
{
  get
  {
    try
    {
      dynamic blocks = Newtonsoft.Json.Linq.JObject.Parse(Contents);
      return blocks.blocks.ToString();
    }
    catch
    {
      return null;
    }
  }
}
```
```
data: {
  blocks:  @Html.Raw(Model.ContentBlocks)
},
```

에디터로 작성된 내용 보기 처리 방법
-

editorjs 에디터는 자체적으로 blocks json을 html로 변환해 주는 기능을 제공해주고 있지 않습니다. 개인적으론 이런 부분이 가장 큰 단점이지 않을까 생각 됩니다.<br/>

readOnly모드가 제공되어 readOnly로 설정해서 뷰 용도로 사용할 순 있는데 readOnly모드에서는 ImageUploader Tool등을 사용할 수 없습니다.
사실상 에디터에서 readOnly모드를 지원하지 않는 tool을 사용하는 경우 사용할 수 없는 방법 입니다.

에디터 json html 변환 방법 (View)
-

그나마 git을 찾아보면 editorjs json을 html로 변환해 주는 parser 모듈이 몇개가 있습니다.<br/>
처리 방법은 모두 순수 blocks json을 하나씩 파싱해 일일히 해당 block에 맞는 html로 변환시켜 주는 방법 입니다.<br/>
이런 방법으로 에디터의 json데이터를 html로 변환 후 뷰 용도로 처리할 수 있습니다.<br/>

**[ContentsView.cshtml]**
```html
@model CoreNote.Models.Archive
@{
    ViewData["Title"] = "Article";
}
<div class="row">
  <div class="col-sm-2">
    <p>제목</p>
  </div>
  <div class="col-sm-10">
    <p>@Model.Title</p>
  </div>
</div>

<hr style="border: 2px solid gray; border-radius: 5px;" />

<div class="row" style="width: 100%;">
  <p class="card">
  <table class="table table-bordered" style="font-size: 1rem;">
    <tbody>
      <tr>
        <td id="Content">
        </td>
      </tr>
    </tbody>
  </table>
  </p>
</div>
```

일단 View페이지를 만들고 저장된 blocks json을 html로 변환해서 Content <td>태그에 출력시켜 보도록 하겠습니다.<br/>
**[ContentsView.cshtml]**
```js
<script>
  const blocks = @Html.Raw(Model.Contents)
  let html = '';
  blocks.blocks.forEach(function(block) {
    switch (block.type) {
    case 'header':
      html += `<h${block.data.level}>${block.data.text}</h${block.data.level}>`;
      break;
    case 'paragraph':
      html += `<p>${block.data.text}</p>`;
      break;
    case 'delimiter':
      html += '<hr />';
      break;
    case 'image':
      html += `<img class="img-fluid" src="${block.data.file.url}" title="${block.data.caption}" /><br /><em>${block.data.caption}</em>`;
      break;
    case 'list':
      html += '<ul>';
      block.data.items.forEach(function(li) {
        html += `<li>${li}</li>`;
      });

      html += '</ul>';
      break;
    default:
      console.log('Unknown block type', block.type);
      console.log(block);
    break;
  }
  
  document.getElementById('Content').innerHTML = html;
  
});
</script>
```
  
이렇게 block의 type에 맞게 html로 변환시켜 Content <td>태그에 출력할 수 있습니다.
  
위 방법에서 조금 더 확장 된 형태로 Parser를 제공하는 모듈이 있습니다. [https://github.com/miadabdi/editorjs-parser](https://github.com/miadabdi/editorjs-parser)<br/>
위 모듈에서 현재(2022. 04. 18) 지원하는 block은 다음과 같습니다.<br/>
- Paragraph
- Header
- Table
- Raw
- Delimiter
- Code
- Quote
- List
- Embed
- Image
- Simple-image

추가로 editorjs-parser 모듈에서 지원되지 않은 block은 커스텀하게 추가해서 사용할 수 있습니다.<br/>

edjsParser의 초기화 형태는 다음과 같이 정의 되어 있습니다.<br/>
```js
const parser = new edjsParser(config, customParsers, embedMarkup);
```
사용 방법은 다음과 같이 처리 할 수 있습니다.

**[ContentsView.cshtml]**
```js
<script src="https://cdn.jsdelivr.net/npm/editorjs-parser@1/build/Parser.browser.min.js"></script><!-- EditorJs Parser -->
<script>
  const jsonData = @Html.Raw(Model.Contents);
  
  // https://github.com/miadabdi/editorjs-parser
  const parser = new edjsParser({
    embed: { useProvidedLength: false },
    quote: { applyAlignment: true },
  },
  {
    linkTool: function(data, config) {
        return `<a href="${data.link}" target="_blank">${data.link}</a>`;
    },
    attaches: function(data, config) {
        return `<a href="${data.file.url}" target="_blank">${data.title}(size : ${data.file.size} kb)</a>`;
    },
    delimiter: function(data, config) {
        return '<hr/>';
    },
  },
  {
    youtube: `<iframe src="<%data.embed%>" width="<%data.width%>"><%data.caption%></iframe>`,
  });
  
  const markup = parser.parse(jsonData);

  document.getElementById('Content').innerHTML = markup;
</script>
```
  
해당 Parser에서 linkTool, arraches를 지원하지 않아 새로 추가 하였으며, delimiter block은 기존에 &lt;br/&gt;로 처리 되던 것을 &lt;hr/&gt;로 오버라이딩 처리 했습니다.<br/>
이렇게 해서 editorjs 에디터 적용 방법을 알아 봤습니다.

{% include content_adsense.html %}
