#iOS文件系统	Working with File System

-------

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


##应用程序沙盒机制
在iOS中处于安全的原因，应用程序访问文件系统受到了一定的限制，这个限制称为**沙盒机制**（SandBox）。在文件系统中，你的应用程序只能在一个特定的目录（Home目录）中进行读取、创建、删除操作，这个目录是你应用程序安装目录，目录的位置由iOS系统指定。你应用程序本身以及所有相关的数据都全部储存在这个目录中。Xamarin.iOS提供了不少属性和方法来管理这个目录中的文件和文件夹。

###应用程序包
**Application Bundle**将你的应用程序囊括在一个文件夹中，它与其他文件夹的区别在于目录名后加上了.app的后缀。Bundle中包含了应用程序的可执行文件以及所有的相关内容（图片、布局文件、资源文件等等）。

在Mac系统中查看Bundle目录时，系统会隐藏掉.app后缀名，虽然他和别的目录看起来不一样，但是实际操作是一样的。
需要查看**Application Bundle**目录你只需要在Xamarin.Studio中右击项目选择`Open Containing Folder`，并打开`bin\Debug\`目录就能看到一个应用程序的图标，如下图。
![Bundle](http://7xiiiw.com1.z0.glb.clouddn.com/githubFileSystem_Bundle.png)


在图标右键选择`显示包内容(View Package Contents)`就可以看到**Application Bundle**中的内容。

![enter image description here](http://7xiiiw.com1.z0.glb.clouddn.com/githubFileSystem_Bundle_2.png)

 **Application Bundle**会被安装在模拟器以及调试时真机上，最终提交到App Store时也是提交的**Application Bundle**。

###应用程序目录
在应用程序安装到设备上时，系统会指定一个目录作为应用程序的Home目录，并将Application Bundle放到这个目录当中。你的应用程序可以读取这个目录中的内容，但不应该对Home目录的根目录做任何的修改，任何修改可能使应用程序启动失败。
在iOS7以及更早的版本中，你可以在在应用程序的根目录中创建文件夹，但是在iOS8中，应用程序的根目录是不能访问和修改的。
下面列举出了根目录中一些子目录的作用。
 
>* [应用程序名称].app
- 在iOS7以及更早的版本中，Application Bundle以及应用的可执行文件都位于这里。在Xamarin Studio项目中生成操作被标记为`Bundle Resource`的文件也位于这个目录。
 
 <Br/>
 
>* Documents
- 这个目录用来储存用户的文档以及部分应用程序的数据文件。
- 这个目录中的文件可以通过iTunes共享出去。默认情况下是禁用了共享功能的，将`UIFileSharingEnabled`字段添加到 `Info.plist `文件中，以开启应用程序的iTunes文件共享。
- 即使应用程序不开启共享，也应该避免在这个目录中存储你不希望用户看到的文件（如数据库文件，除非你想要分享他们）。
- 你可以通过` Environment.GetFolderPath (Environment.SpecialFolder.MyDocuments)`方法来获取应用程序所属的Document目录的路径。
- 通过iTunes备份时，Documents目录中所有的文件以及文件夹都将被备份。
 
  <Br/>
  
>* Library
- Library目录主要负责储存不是由用户创建的文件，如数据库、缓存等。用户也无法通过iTunes访问到Library目录。
- 你可以在Library目录中创建自己的文件夹，但是Library中已经有一些由系统创建好的文件夹`Preferences`、`Caches`。
- 通过iTunes备份时，Library目录中除了Caches之外的所有目录，包括你自己创建的目录都将被备份。
 
  <Br/>
  
>* Library/Preferences/
- 应用程序的偏好设置将会储存在此目录中，不要自己访问这个目录，而是通过`NSUserDefaults`类来控制读取和修改偏好设置。
- 通过iTunes备份时，Library/Preferences将会被备份。
 
  <Br/>
  
>* Library/Caches/
- Caches目录通常用来储存数据缓存，因为Caches文件有可能会被清空，iTunes备份时也不会备份，所以应用程序在失去Cache后应该可以轻松的重新建立自己的缓存。

  <Br/>
  
>* tmp/
- tmp目录主要储存临时文件。临时文件使用完后请删除，以节省设备的储存空间。


###使用代码访问相关文件夹
下面这段代码演示了如何访问`Documents`目录，并在其中的`Library`目录中创建了一个文本文件。
```csharp
var documents = Environment.GetFolderPath (Environment.SpecialFolder.MyDocuments);
var library = Path.Combine (documents, "..", "Library");
var filename = Path.Combine (library, "WriteToLibrary.txt");
File.WriteAllText(filename, "Write this text into a file in Library");
```

创建文件夹也同样方便：
```csharp
var documents = Environment.GetFolderPath (Environment.SpecialFolder.MyDocuments);
var library = Path.Combine (documents, "..", "Library");
var directoryname = Path.Combine (library, "NewLibraryDirectory");
Directory.CreateDirectory(directoryname);
```
要获取 `Cache` 和 `tmp` 目录的路径可以通过如下的方式：
```csharp
var documents = Environment.GetFolderPath (Environment.SpecialFolder.MyDocuments);
var cache = Path.Combine (documents, "..", "Library", "Caches");
var tmp = Path.Combine (documents, "..", "tmp");
```


###通过iTunes共享文件
用户可以通过iTunes来访问我们应用程序的`Document`文件夹中的文件。要开启这项功能我们需要在`Info.plist`文件中增加`Application supports iTunes sharing (UIFileSharingEnabled)`选项，双击`Info.plist`文件，并进入`Source`视图（左下角）:
![Info](http://7xiiiw.com1.z0.glb.clouddn.com/github09_UIFileSharingEnabled_plist.png)

当设备连接到iTunes时，可以在iTunes中访问这些文件：
![iTunes](http://7xiiiw.com1.z0.glb.clouddn.com/github10_iTunes_File_Sharing.png)

用户只能在iTunes中看到`Document`目录的顶级目录中的文件，不能查看到子目录中的内容（不过可以复制出来）。例如GoodReader就是通过这种方式让用户可以将自己的EPUB、PDF文件复制到设备上。

用户有可能在对`Document`目录进行破坏性的修改，程序应该要考虑到用户不小心使`Document`中的文件损坏时的情况，在必要的情况下能重新生成`Document`目录。


###备份与恢复
当设备通过iTunes恢复时，应用程序的Home目录中所有的目录都将会被恢复，除了一下特殊说明的目录：
>* [应用程序名称].app
- 通常情况下的[应用程序名称].app是可以恢复的，不过如果你修改过Bundle的话，被修改的Bundle是不会被恢复的。所以在程序安装完成后不要对Bundle做任何改动。
>* Library/Caches和tmp
- 这2个都是缓存和临时目录，iTunes不会对这2个目录进行备份。

备份大量的数据需要很多空间和时间，所以如果可以由程序生成或者可以通过网络获取的数据请尽量放在Caches和tmp目录里面。

需要注意的是，当iOS设备储存容量不足时，系统有可能会删除没在运行中应用程序的Cache和tmp目录中的内容。