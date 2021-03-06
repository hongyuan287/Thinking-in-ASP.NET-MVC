
# ASP.NET MVC 随想录（5）——创建 ASP.NET MVC Bootstrap Helpers

> ASP.NET MVC 允许开发者创建自定义的 HTML Helpers，不管是使用静态方法还是扩展方法。一个HTML Helper 本质上其实是输出一段 HTML 字符串。
> HTML Helpers 能让我们在多个页面上公用同一段 HTML 标记，这样不仅提高了稳定性也便于开发者去维护。当然对于这些可重用的代码，开发者也方便对他们进行单元测试。所以，创建 ASP.NET MVC Bootstrap Helpers 是及其有必要的。

## 内置的HTML Helpers

ASP.NET MVC 内置了若干标准 HTML Helpers，通过 `@ HTML` 来调用这些方法在视图引擎中解析、渲染输出 HTML 内容，这允许开发者在多个视图中重用公共的方法。

举个例子，以下代码产生一个 type 等于 text 的 Input ，并且其 id 和 name 都等于 CustomerName，其 Value 等于 Northwind Traders：

    @ Html.TextBox("CustomerName","Northwind Traders");

大多数内置的 HTML helpers 提供传入匿名类型为元素产生指定 HTML 属性的选项，对上述的 @ HTML.TextBox方法稍作修改，通过传入匿名类型设置输出元素的 style 属性：

     @Html.TextBox("CustomerName","Northwind Traders", new { style="background-color:Blue;" })

## 创建自定义的 Helpers

因为 Bootstrap 提供了大量不同的组件，所以创建 Bootstrap helpers 可以在多个视图上快速使用这些组件。在 ASP.NET MVC 中最简单创建 Bootstrap helpers 是通过 **@helper** 语法来实现。一个自定义的 helper 可以包含任何 HTML 标记甚至 Razor 标记，你可以通过如下步骤来创建：

在项目的根目录创建文件夹 App_Code  
在 App_Code 文件夹中新建 BootstrapHelpers.cshtml 文件并加入如下代码

    @helper PrimaryButtonSmall(string id,string caption)
    {
    	<button id="@id" type="button" class="btn btn-primary btn-sm">@caption</button>
    }

上述代码使用 **@helper** 创建了一个新的名为 PrimaryButtonSmall helper，它接受 2个参数，分别是 Id 和 caption。其中，它产生一个 Button 类型的 HTML 标记并设置了 Bootstrap 的样式。  

**注意：任何自定义的 helpers 必须存在 App_Code 文件夹中，这样才能被 ASP.NET MVC 视图识别。**

* 在视图中通过 @BootstrapHelpers.PrimaryButtonSmall("btnSave","保存")来使用新创建的helper。
* 它将产生如下 Bootstrap HTML 元素：

![](images/Chapter5/2.png)

当然，为了让我们的 helper 更加通用性，比如指定大小、样式等，对上述稍作如下修改，增加传入的参数：  

    @helper Button(string style, string size, string caption, string id)
    {
    	<button id="@id" type="button" class="btn btn-@style btn-@size">@caption </button>
    }

现在我们可以这样去使用：

    @BootstrapHelpers.Button("danger","lg","危险","btnDanger")

它将产生如下样式的按钮：

![](images/Chapter5/1.png)

不过，这种方式的 helper 唯一的不足是你需要"hard code"传入样式和尺寸，这可能需要你非常熟悉Bootstrap 的样式。

## 使用静态方法创建 Helpers

通过静态方法同样也能快速方便的创建自定义 Bootstrap helpers，同样它也是返回了 HTML 标记，要创建静态方法，你可以按照如下步骤来实现：

1.添加命了 Helpers 的文件夹  
2.创建如下枚举类    

    public class ButtonHelper
       {
       public static MvcHtmlString Button(string caption, Enums.ButtonStyle style, Enums.ButtonSize size)
       {
           if (size != Enums.ButtonSize.Normal)
           {
               return new MvcHtmlString(string.Format("<button type=\"button\" class=\"btn btn-{0} btn-{1}\">{2}</button>", style.ToString().ToLower(), ToBootstrapSize(size), caption));
           }
           return new MvcHtmlString(string.Format("<button type=\"button\" class=\"btn btn-{0}\">{1}</button>", style.ToString().ToLower(), caption));
       }
 
       private static string ToBootstrapSize(Enums.ButtonSize size)
       {
           string bootstrapSize = string.Empty;
           switch (size)
           {
               case Enums.ButtonSize.Large:
                   bootstrapSize = "lg";
                   break;
 
               case Enums.ButtonSize.Small:
                   bootstrapSize = "sm";
                   break;
 
               case Enums.ButtonSize.ExtraSmall:
                   bootstrapSize = "xs";
                   break;
           }
           return bootstrapSize;
       }
      }
    

Button 方法返回了一个 MvcHtmlString 对象，它代表了编码过后的 HTML 字符。

1.通过使用静态方法来调用：  

     @ButtonHelper.Button("危险", Enums.ButtonStyle.Danger, Enums.ButtonSize.Large)

你可以和之前的"hard code"写法进行比较，尽管他们产生相同的结果：

    @BootstrapHelpers.Button("danger","lg","危险","btnDanger")

## 使用扩展方法创建Helpers

内置的 ASP.NET MVC helper（**@HTML**）是基于扩展方法的，我们可以再对上述的静态方法进行升级——使用扩展方法来创建 Bootstrap helpers。

1.在 Helpers 文件夹下创建 ButtonExtensions 类
2.修改 ButtonExtensions为Static 类型
3.修改 Namespace为System.Web.Mvc.Html，这样方便 **@HTML** 调用扩展方法
4.添加扩展方法，返回 MvcHtmlString  

     public static MvcHtmlString BootstrapButton(this HtmlHelper helper, string caption, Enums.ButtonStyle style, Enums.ButtonSize size)
        {
            if (size != Enums.ButtonSize.Normal)
            {
                return new MvcHtmlString(string.Format("<button type=\"button\" class=\"btn btn-{0} btn-{1}\">{2}</button>", style.ToString().ToLower(), ToBootstrapSize(size), caption));
            }
            return new MvcHtmlString(string.Format("<button type=\"button\" class=\"btn btn-{0}\">{1}</button>", style.ToString().ToLower(), caption));
       ` }`
因为 BootstrapButton 方法是扩展方法，通过如下方式去调用：

    @Html.BootstrapButton("很危险",Enums.ButtonStyle.Danger,Enums.ButtonSize.Large)

写法虽不同，但返回的结果都是一致的。

## 创建 Fluent Helpers

Fluent Interface（参考：http://martinfowler.com/bliki/FluentInterface.html）用于软件开发实现了一种面向对象的 API，以这种方式，它提供了更多的可读性代码，易于开发人员理解。通常通过链式编程来实现。

举个例子，我们将创建一个 HTML helper 来产生一个可关闭的警告框，使用 Fluent Interface 可以这样来调用：  

     @Html.Alert("警告").Warning().Dismissible()

所以要创建 Fluent helpers，需要实现如下步骤：

1.创建 IFluentAlert 实现 IHtmlString 接口，这是非常重要的一步，对于 ASP.NET MVC Razor 视图引擎，如果 @ 之后返回的类型实现了 **IHtmlString** 接口，那么视图引擎会**自动**调用 ToHtmlString() 方法，返回实际的 HTML 标记。
  
    public interface IAlertFluent : IHtmlString 
    {
    
       IAlertFluent Dismissible(bool canDismiss = true);
    
    }

2.创建 IAlert 实现 IFluentAlert 接口  

    public interface IAlert : IAlertFluent
    {
    
	    IAlertFluent Danger();
	    
	    IAlertFluent Info();
	    
	    IAlertFluent Success();
	    
	    IAlertFluent Warning();
    
    }

 3.创建 Alert 继承 IAlert 接口  

    public class Alert : IAlert
       {
       private Enums.AlertStyle _style;
       private bool _dismissible;
       private string _message;
 
       public Alert(string message)
       {
           _message = message;
       }
 
       public IAlertFluent Danger()
       {
           _style = Enums.AlertStyle.Danger;
           return new AlertFluent(this);
       }
 
       public IAlertFluent Info()
       {
           _style = Enums.AlertStyle.Info;
           return new AlertFluent(this);
       }
 
       public IAlertFluent Success()
       {
           _style = Enums.AlertStyle.Success;
           return new AlertFluent(this);
       }
 
       public IAlertFluent Warning()
       {
           _style = Enums.AlertStyle.Warning;
           return new AlertFluent(this);
       }
 
       public IAlertFluent Dismissible(bool canDismiss = true)
       {
           this._dismissible = canDismiss;
           return new AlertFluent(this);
       }
 
       public string ToHtmlString()
       {
           var alertDiv = new TagBuilder("div");
           alertDiv.AddCssClass("alert");
           alertDiv.AddCssClass("alert-" + _style.ToString().ToLower());
           alertDiv.InnerHtml = _message;
 
           if (_dismissible)
           {
               alertDiv.AddCssClass("alert-dismissable");
               alertDiv.InnerHtml += AddCloseButton();
           }
 
           return alertDiv.ToString();
       }
 
       private static TagBuilder AddCloseButton()
       {
           var closeButton = new TagBuilder("button");
           closeButton.AddCssClass("close");
           closeButton.Attributes.Add("data-dismiss", "alert");
           closeButton.InnerHtml = "&times;";
           return closeButton;
       }
       }


上述代码中，通过 **TagBuilder** 可以快速的创建 HTML 元素。

4.创建 AlertFluent 继承 IAlertFluent   

    public class AlertFluent : IAlertFluent
    {
        private readonly Alert _parent;
 
        public AlertFluent(Alert parent)
        {
            _parent = parent;
        }
 
        public IAlertFluent Dismissible(bool canDismiss = true)
        {
            return _parent.Dismissible(canDismiss);
        }
 
        public string ToHtmlString()
        {
            return _parent.ToHtmlString();
        }
    }

通过构建这种 Fluent API，我们可以链式的去创建 Bootstrap 组件，这对于不熟悉 Bootstrap Framework 的人来说是非常方便的，我们可以使用 **@HTML.Alert("Title").Danger().Dismissible()** 来创建如下风格的警告框：

![](images/Chapter5/3.png)

## 创建自动闭合的 Helpers

在 ASP.NET MVC 中，内置的 `@HTML.BeginForm()` helper 就是一个自动闭合的 helper。当然我们也能自定义自动闭合的 helpers，只要实现 IDisposable 接口即可。使用 IDisposable 接口，当对象 Dispose 时我们输出元素的闭合标记，具体按照如下步骤：

1.所以在 Helpers 文件夹下创建一个名为 Panel 的文件夹
2.添加 Panel，并实现 IDisposable 接口  

    public class Panel : IDisposable
    {
        private readonly TextWriter _writer;
 
        public Panel(HtmlHelper helper, string title, Enums.PanelStyle style = Enums.PanelStyle.Default)
        {
            _writer = helper.ViewContext.Writer;
 
            var panelDiv = new TagBuilder("div");
            panelDiv.AddCssClass("panel-" + style.ToString().ToLower());
            panelDiv.AddCssClass("panel");
 
            var panelHeadingDiv = new TagBuilder("div");
            panelHeadingDiv.AddCssClass("panel-heading");
 
            var heading3Div = new TagBuilder("h3");
            heading3Div.AddCssClass("panel-title");
            heading3Div.SetInnerText(title);
 
            var panelBodyDiv = new TagBuilder("div");
            panelBodyDiv.AddCssClass("panel-body");
 
            panelHeadingDiv.InnerHtml = heading3Div.ToString();
 
            string html = string.Format("{0}{1}{2}", panelDiv.ToString(TagRenderMode.StartTag), panelHeadingDiv, panelBodyDiv.ToString(TagRenderMode.StartTag));
            _writer.Write(html);
        }
 
        public void Dispose()
        {
            _writer.Write("</div></div>");
        }
    }

上述代码中利用Write属性可以在当前视图中输出HTML标记，并在Dispose方法里输出2个闭合的div标签。

注意，我们重写了TagBuilder的ToString()方法，只让它生成div元素的开始标签。  
3.在View中使用自动闭合的helpers  

    @using (Html.Panel("Title", Enums.PanelStyle.Success))
    {
	    <p>这是自动闭合的Helpers</p>
	    <p>panel..</p>
    }
产生的结果如下：  

![](images/Chapter5/4.png)
## 小结 ##

> 在这篇博客中，为了减少书写HTML标记，我们创建了若干Bootstrap helpers来实现。这些helpers的意义在于能让不了解Bootstrap Framework的人也能快速上手Bootstrap。