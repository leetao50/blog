---
title: 在Asp.Net Core中上传文件
date: 2023-06-13 11:25:44
tags:
---
ASP.NET Core 支持使用缓冲模型绑定（针对较小文件）和无缓冲流式传输（针对较大文件）上传一个或多个文件。

# 小型和大型文件
小型和大型文件的定义取决于可用的计算资源。 应用应对存储方法进行基准测试，以确保它可以处理预期的大小。 基准内存、CPU、磁盘和数据库性能。

虽然无法针对部署的“小”与“大”提供特定边界，但以下是 AspNetCore 针对 FormOptions 的一些相关默认值：
+ 默认情况下， HttpRequest.Form 不会缓冲整个请求正文 (BufferBody) ，但会缓冲包含的任何多部分表单文件。
+ MultipartBodyLengthLimit 是缓冲表单文件的最大大小，默认值为 128MB。
+ MemoryBufferThreshold 指示在转换为磁盘上的缓冲区文件之前，内存中的文件缓冲量，默认为 64KB。 MemoryBufferThreshold 充当小型和大型文件之间的边界，这些文件根据应用资源和方案而引发或

# 文件上传方案

缓冲和流式传输是上传文件的两种常见方法。

***缓冲***

将整个文件读入 IFormFile。 IFormFile 是用于处理或保存文件的文件的 C# 表示形式。

文件上传使用的磁盘和内存取决于并发文件上传的数量和大小。 如果应用尝试缓冲过多上传，站点就会在内存或磁盘空间不足时崩溃。 如果文件上传的大小或频率会消耗应用资源，请使用流式传输。

会将大于 64 KB 的所有单个缓冲文件从内存移到磁盘的临时文件。

较大请求的临时文件将写入环境变量中 ASPNETCORE_TEMP 名为 的位置。 如果未 ASPNETCORE_TEMP 定义 ，则文件将写入当前用户的临时文件夹。

**流式处理**

从多部分请求收到文件，然后应用直接处理或保存它。 流式传输无法显著提高性能。 流式传输可降低上传文件时对内存或磁盘空间的需求。

# 通过缓冲的模型绑定将小型文件上传到物理存储

要上传小文件，请使用多部分窗体或使用 JavaScript 构造 POST 请求。

~~~JS
<script>
  "use strict";

  function AJAXSubmit (oFormElement) {
    var oReq = new XMLHttpRequest();
    oReq.onload = function(e) { 
    oFormElement.elements.namedItem("result").value = 
      'Result: ' + this.status + ' ' + this.statusText;
    };
    oReq.open("post", oFormElement.action);
    oReq.send(new FormData(oFormElement));
  }
</script>
~~~
如下示例中：

+ 循环访问一个或多个上传的文件。
+ 使用 Path.GetTempFileName 返回文件的完整路径，包括文件名称。
+ 使用应用生成的文件名将文件保存到本地文件系统。
+ 返回上传的文件的总数量和总大小。
~~~C#
public async Task<IActionResult> OnPostUploadAsync(List<IFormFile> files)
{
    long size = files.Sum(f => f.Length);

    foreach (var formFile in files)
    {
        if (formFile.Length > 0)
        {
            var filePath = Path.GetTempFileName();

            using (var stream = System.IO.File.Create(filePath))
            {
                await formFile.CopyToAsync(stream);
            }
        }
    }

    // Process uploaded files
    // Don't rely on or trust the FileName property without validation.

    return Ok(new { count = files.Count, size });
}
~~~

使用 Path.GetRandomFileName 生成文件名（不含路径）。 在下面的示例中，从配置获取路径：

~~~C#
foreach (var formFile in files)
{
    if (formFile.Length > 0)
    {
        var filePath = Path.Combine(_config["StoredFilesPath"], 
            Path.GetRandomFileName());

        using (var stream = System.IO.File.Create(filePath))
        {
            await formFile.CopyToAsync(stream);
        }
    }
}
~~~

传递给 FileStream 的路径必须包含文件名。 如果未提供文件名，则会在运行时引发 UnauthorizedAccessException。

使用 IFormFile 技术上传的文件在处理之前会缓冲在内存中或服务器的磁盘中。 在操作方法中，IFormFile 内容可作为 Stream 访问。 除本地文件系统之外，还可以将文件保存到网络共享或文件存储服务，如 Azure Blob 存储。

> 如果在未删除先前临时文件的情况下创建了 65,535 个以上的文件，则 Path.GetTempFileName 将抛出一个 IOException。 65,535 个文件限制是每个服务器的限制。 有关 Windows 操作系统上的此限制的详细信息，请参阅以下主题中的说明：
> + GetTempFileNameA 函数
> + GetTempFileName

# 使用缓冲的模型绑定将小型文件上传到数据库
要使用实体框架将二进制文件数据存储在数据库中，请在实体上定义 Byte 数组属性：

~~~C#
public class AppFile
{
    public int Id { get; set; }
    public byte[] Content { get; set; }
}

//为包括 IFormFile 的类指定页模型属性：
public class BufferedSingleFileUploadDbModel : PageModel
{
    ...

    [BindProperty]
    public BufferedSingleFileUploadDb FileUpload { get; set; }

    ...
}

public class BufferedSingleFileUploadDb
{
    [Required]
    [Display(Name="File")]
    public IFormFile FormFile { get; set; }
}
~~~
将窗体发布到服务器后，将 IFormFile 复制到流，并将它作为字节数组保存在数据库中。 在下面的示例中，_dbContext 存储应用的数据库上下文：

~~~C#
public async Task<IActionResult> OnPostUploadAsync()
{
    using (var memoryStream = new MemoryStream())
    {
        await FileUpload.FormFile.CopyToAsync(memoryStream);

        // Upload the file if less than 2 MB
        if (memoryStream.Length < 2097152)
        {
            var file = new AppFile()
            {
                Content = memoryStream.ToArray()
            };

            _dbContext.File.Add(file);

            await _dbContext.SaveChangesAsync();
        }
        else
        {
            ModelState.AddModelError("File", "The file is too large.");
        }
    }

    return Page();
}
~~~

# 通过流式传输上传大型文件
~~~C#
public async Task<IActionResult> UploadDatabase()
{
    if (!MultipartRequestHelper.IsMultipartContentType(Request.ContentType))
    {
        ModelState.AddModelError("File", 
            $"The request couldn't be processed (Error 1).");
        // Log error

        return BadRequest(ModelState);
    }

    // Accumulate the form data key-value pairs in the request (formAccumulator).
    var formAccumulator = new KeyValueAccumulator();
    var trustedFileNameForDisplay = string.Empty;
    var untrustedFileNameForStorage = string.Empty;
    var streamedFileContent = Array.Empty<byte>();

    var boundary = MultipartRequestHelper.GetBoundary(
        MediaTypeHeaderValue.Parse(Request.ContentType),
        _defaultFormOptions.MultipartBoundaryLengthLimit);
    var reader = new MultipartReader(boundary, HttpContext.Request.Body);

    var section = await reader.ReadNextSectionAsync();

    while (section != null)
    {
        var hasContentDispositionHeader = 
            ContentDispositionHeaderValue.TryParse(
                section.ContentDisposition, out var contentDisposition);

        if (hasContentDispositionHeader)
        {
            if (MultipartRequestHelper
                .HasFileContentDisposition(contentDisposition))
            {
                untrustedFileNameForStorage = contentDisposition.FileName.Value;
                // Don't trust the file name sent by the client. To display
                // the file name, HTML-encode the value.
                trustedFileNameForDisplay = WebUtility.HtmlEncode(
                        contentDisposition.FileName.Value);

                streamedFileContent = 
                    await FileHelpers.ProcessStreamedFile(section, contentDisposition, 
                        ModelState, _permittedExtensions, _fileSizeLimit);

                if (!ModelState.IsValid)
                {
                    return BadRequest(ModelState);
                }
            }
            else if (MultipartRequestHelper
                .HasFormDataContentDisposition(contentDisposition))
            {
                // Don't limit the key name length because the 
                // multipart headers length limit is already in effect.
                var key = HeaderUtilities
                    .RemoveQuotes(contentDisposition.Name).Value;
                var encoding = GetEncoding(section);

                if (encoding == null)
                {
                    ModelState.AddModelError("File", 
                        $"The request couldn't be processed (Error 2).");
                    // Log error

                    return BadRequest(ModelState);
                }

                using (var streamReader = new StreamReader(
                    section.Body,
                    encoding,
                    detectEncodingFromByteOrderMarks: true,
                    bufferSize: 1024,
                    leaveOpen: true))
                {
                    // The value length limit is enforced by 
                    // MultipartBodyLengthLimit
                    var value = await streamReader.ReadToEndAsync();

                    if (string.Equals(value, "undefined", 
                        StringComparison.OrdinalIgnoreCase))
                    {
                        value = string.Empty;
                    }

                    formAccumulator.Append(key, value);

                    if (formAccumulator.ValueCount > 
                        _defaultFormOptions.ValueCountLimit)
                    {
                        // Form key count limit of 
                        // _defaultFormOptions.ValueCountLimit 
                        // is exceeded.
                        ModelState.AddModelError("File", 
                            $"The request couldn't be processed (Error 3).");
                        // Log error

                        return BadRequest(ModelState);
                    }
                }
            }
        }

        // Drain any remaining section body that hasn't been consumed and
        // read the headers for the next section.
        section = await reader.ReadNextSectionAsync();
    }

    // Bind form data to the model
    var formData = new FormData();
    var formValueProvider = new FormValueProvider(
        BindingSource.Form,
        new FormCollection(formAccumulator.GetResults()),
        CultureInfo.CurrentCulture);
    var bindingSuccessful = await TryUpdateModelAsync(formData, prefix: "",
        valueProvider: formValueProvider);

    if (!bindingSuccessful)
    {
        ModelState.AddModelError("File", 
            "The request couldn't be processed (Error 5).");
        // Log error

        return BadRequest(ModelState);
    }

    // **WARNING!**
    // In the following example, the file is saved without
    // scanning the file's contents. In most production
    // scenarios, an anti-virus/anti-malware scanner API
    // is used on the file before making the file available
    // for download or for use by other systems. 
    // For more information, see the topic that accompanies 
    // this sample app.

    var file = new AppFile()
    {
        Content = streamedFileContent,
        UntrustedName = untrustedFileNameForStorage,
        Note = formData.Note,
        Size = streamedFileContent.Length, 
        UploadDT = DateTime.UtcNow
    };

    _context.File.Add(file);
    await _context.SaveChangesAsync();

    return Created(nameof(StreamingController), null);
}
~~~