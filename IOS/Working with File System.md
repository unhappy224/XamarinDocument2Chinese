#iOS文件系统	Working with File System

-------
[TOC]

Xamarin.iOS操作iOS文件系统的方法与你在其他.net应用程序中一样，都是使用System.IO命名空间下的类型。
我们需要注意的是，iOS对文件的IO操作有一定的限制以及提供了一些特殊功能的目录。
本文将介绍这些限制以及功能，并演示如何在Xamarin.iOS应用程序中对文件进行操作。

##预览
你可以在iOS中正常使用`System.IO`命名空间下的类型，用法与其他 .net 应用程序没任何区别。例如`File`类允许你创建、删除和读取文件。`Directory`类型允许你创建、删除以及获取一个文件夹中所有的内容。同样的你可以使用`Stream`以及他的那些子类来操作各种数据流。

iOS为了保护用户数据以及防范恶意App，对程序的IO操作做出了一些限制。这些限制主要来自于沙盒机制。**沙盒机制**（Application Sandbox)是一个规则集合，他规定了我们如何访问文件、设置、网络等资源。应用程序的读写权限被限制在他的安装目录当中，除了本身的安装目录外，应用程序无法访问其他程序的文件。

iOS也有一些特殊的文件夹：一些文件夹在升级和备份的时候需要特殊对待，应用程序也可以通过这些特殊文件夹来与iTunes共享文件。

本文会详细讨论iOS文件系统的特殊功能以及限制，并通过一个简单的例子来演示如何使用iOS文件系统的功能。
![演示程序](http://7xiiiw.com1.z0.glb.clouddn.com/iOS_FileSystem_01.png)

##常规的文件操作
Xamarin.iOS允许你在iOS中使用 .Net `System.IO` 中的类来进行文件操作。下面的代码段可以在本文的演示程序中的`SampleCode.cs` 中。

###文件夹操作
这段代码将打印出当前应用程序执行目录（由**"./"**参数指定)中的所有的子文件夹。这个目录里面包含了应用程序部署展开后所有的文件和文件夹。（将会输出在 [视图]-[输出] 窗口中）。

```csharp
var directories = Directory.EnumerateDirectories("./");
foreach (var directory in directories) {
      Console.WriteLine(directory);
}
```

###读取文件
读取文件只需要一行代码。下面这个例子将在输出窗口打印出文本文件的内容。
```csharp
var text = File.ReadAllText("TestData/ReadMe.txt");
Console.WriteLine(text);
```

####XML序列化
Xml的序列化与反序列化都与 .Net 其他平台一样，都是使用`System.Xml`命名空间提供的类，So Easy！
```csharp
using (TextReader reader = new StreamReader("./TestData/test.xml")) {
      XmlSerializer serializer = new XmlSerializer(typeof(MyObject));
      var xml = (MyObject)serializer.Deserialize(reader);
}
```
更多关于序列化的信息请参阅MSDN中的 [`System.Xml`](https://msdn.microsoft.com/en-us/library/system.xml.aspx)一节

###创建文件与文件夹
这段代码演示了如何使用`Environment`类来访问Document文件夹，按照沙盒机制的要求我们只能在Document文件夹中创建文件和文件夹。
```csharp
var documents =
 Environment.GetFolderPath (Environment.SpecialFolder.MyDocuments); 
var filename = Path.Combine (documents, "Write.txt");
File.WriteAllText(filename, "Write this text into a file");
```
创建文件夹的方法与创建文件的方法差不多。
```csharp
var documents =
 Environment.GetFolderPath (Environment.SpecialFolder.MyDocuments);
var directoryname = Path.Combine (documents, "NewDirectory");
Directory.CreateDirectory(directoryname);
```
更多关于`System.IO`命名空间的更多信息请参阅[MSDN文档](https://msdn.microsoft.com/en-us/library/system.io.aspx)

###Json序列化
在Xamarin.iOS中处理Json数据需要用到Json.NET库。我们可以通过Nuget来获取Json.NET。

![Image2](http://7xiiiw.com1.z0.glb.clouddn.com/githubjson01.png)

接下来我们需要定义需要序列化/反序列化的数据模型类（`Account.cs`）。

```csharp
using System;
using System.Collections.Generic;

namespace FileSystem
{
    public class Account
    {
        #region Computed Properties
        public string Email { get; set; }
        public bool Active { get; set; }
        public DateTime CreatedDate { get; set; }
        public List<string> Roles { get; set; }
        #endregion

        #region Constructors
        public Account() {

        }
        #endregion
    }
}
```

最后创建一个`Account`类型的对象，序列化并将结果写入文件中。

```csharp
// Create a new record
var account = new Account(){
    Email = "monkey@xamarin.com",
    Active = true,
    CreatedDate = new DateTime(2015, 5, 27, 0, 0, 0, DateTimeKind.Utc),
    Roles = new List<string> {"User", "Admin"}
};

// Serialize object
var json = JsonConvert.SerializeObject(account, Newtonsoft.Json.Formatting.Indented);

// Save to file
var documents = Environment.GetFolderPath (Environment.SpecialFolder.MyDocuments);
var filename = Path.Combine (documents, "account.json");
File.WriteAllText(filename, json);
```
更多关于Json.NET的信息请参阅[相关文档](http://www.newtonsoft.com/json/help/html/Introduction.htm)

###注意事项
尽管Xamarin.iOS与.Net文件操作非常相似，但是在iOS中有一些与.Net平台不同的地方需要特别注意。

#### • 在程序运行时使用项目中文件
在默认情况下，如果你添加到项目中的文件不会包含在最终生成的程序集中，因此在程序运行时不能使用这些文件。为了在程序运行中使用这些文件，你需要在**[ 属性 ]**对话框中将这些文件的**生成操作**（Build Action）修改为**内容**（Content）。

#### • 区分大小写
在iOS中，iOS的文件系统是严格区分大小写的。"ReadMe.txt"和"readMe.txt"是完全不同的文件。
不过iOS设备的模拟器中是不区分大小写的，所以如果你不严格区分大小写的话，程序有可能在模拟器中可以运行，但是在真机中无法运行。

#### • 路径分隔符
iOS使用正斜杠 `'/'`作为分隔符，这与Windows中使用反斜杠`'\'`不一样。
因为这个区别，我们推荐使用`System.IO.Path.Combine()`函数来连接路径，而不是使用硬编码特定的路径分隔符，以保证代码的跨平台性。

