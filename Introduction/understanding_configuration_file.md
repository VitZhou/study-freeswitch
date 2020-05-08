# 了解配置文件

#### 关于

FreeWITCH使用XML文件来配置核心及所有模块. 所以配置文件可能会变得非常大且复杂. 默认配置为新用户提供了一个较好的起点.如果你想直接跳过并开始使用FreeSWITH. 请参考[配置FreeSWITCH](https://freeswitch.org/confluence/display/FREESWITCH/Configuring+FreeSWITCH),了解默认配置的quick overview以及开始使用时需要做的更改.但是如果你想用FreeSIWTH做任何产品,我们强烈建议你花些时间深入了解配置文件的工作原理.在这一页中,我们将对配置系统做一个深入的概述.

## XML

配置文件是用XML编写的,如果你是个XML的新手,请查看XML基础知识.请参考下面的Modules部分.该配置被命名为freeswitch.xml,默认情况下,它位于/etc/freeswitch目录,你也可以在命令行参数中指定另一个文件路径。

这个xml文件被分为多个部分,如下图所示,在配置文件的基本模式中可以看到。每个部分都有它自己的子节和模式，在下面的章节中，我们将探讨每个单独部分的模式。

```xml
<document type="freeswitch/xml">
  <section name="configuration" description="Various Configuration">
    <!-- Module configuration goes here -->
  </section>  
  <section name="dialplan" description="Regex/XML Dialplan">
    <!-- Instructions on how to route incoming calls goes here -->
  </section>
  <section name="chatplan" description="Regex/XML Chatplan">
    <!-- Instructions on how to route incoming SMS messages -->
  </section>
  <section name="directory" description="User Directory">
    <!-- User configuration goes here --> 
  </section>
  <section name="languages" description="Language Management">
    <!-- Here you can configure support for playing the system prompts in different languages -->
  </section>
</document>
```

## 配置

在配置部分，您可以配置FreeSWITCH核心和大多数模块。在配置部分中有许多配置元素，通常每个模块都有一个。大多数配置元素使用以下XML模式。

```xml
<section name="configuration" description="Various Configuration">
    <configuration Name="mymodule.conf" Description="My Module Configuration">
        <settings>
            <param name="my_param" value="my value" />
        </settings>
    </configuration>
    <configuration Name="another_module.conf" Description="Other Module Configuration">
        <settings>
            <param name="my_param" value="my value" />
        </settings>
    </configuration>
</section>
```

某些模块使用不同的XML元素和/或额外的XML属性，请参考每个模块的confluence页面，了解其使用的具体参数、元素和属性的详细信息。

除了个别模块的配置元素外，还有2个特殊的配置元素用于配置FreeSWITCH内核本身

#### switch.conf

这是您为 FreeSWITCH 核心配置设置的地方。除了个别模块的配置元素外，还有2个特殊的配置元素用于配置FreeSWITCH内核本身。

#### modules.conf

本节告诉FreeSWITCH在启动时要加载哪些模块。您可以随时从命令行中加载其他模块。更多信息请参见[模块配置页面](https://freeswitch.org/confluence/display/FREESWITCH/XML+Modules+Configuration)。FreeSWITCH启动时，首先加载switch.conf，然后是modules.conf。当每个模块被加载时，FreeSWITCH将解析并加载该模块使用的配置元素。

## 拨号计划

拨号计划部分是你设置所有呼叫路由规则的地方.

## 聊天计划

在这里你可以为短信和其他聊天工具设置路由规则。

### languages

预处理指令

XML文件是纯文本，没有XML内置的编程逻辑功能。虽然它有易读的优点，但也有一些缺点。

1. 对于像FreeSWITCH这样的大型灵活的系统，配置文件可能是巨大的。默认的配置总长度接近15,000行。
2. 如果您在配置中的许多地方有特殊值要使用，您必须手动复制。当你想改变它时，这可能会造成问题。
3. 你不能运行脚本来动态计算配置的值。

为了解决这些问题，FreeSWITCH使用了自定义的预处理器指令。有一个预处理引擎，它在原始配置文件上运行，并使用预处理指令的结果创建一个新的配置文件。处理后的文件存储在日志目录下（默认是/var/log/freeswitch），命名为 freeswitch.xml.fsxml，这是FreeSWITCH实际加载的文件。预处理程序会在启动时以及重新加载配置文件时运行。如果您在使用预处理程序指令时遇到问题，查看处理后的文件会有帮助。

有2种方法可以指定预处理程序指令

- 定制的X-PRE-PROCESS XML标签
- 在一个XML注释中，你可以用#作为指令的前缀

```xml
<!--Using the X-PRE-PROCESS custom XML tag-->
<X-PRE-PROCESS cmd="set" data="my_variable='value'"/>
<!--Using an XML comment with a #-->
<!--#set my_variable='value'-->
```

标准的XML注释对预处理命令没有影响。如果你想注释一个预处理命令，你可以用X-NO-PROCESS替换成X-PRE-PROCESS

```xml
<!--This is a standard XML comment, the Pre-Process command WILL be processed--> 
<!--<X-PRE-PROCESS cmd="set" data="my_global_var='some value'"/>-->
<!--This pre-process command will NOT run-->
<X-NO-PRE-PROCESS cmd="set" data="my_global_var='some value'"/>
```

以下是所有可用的预处理指令。其中，include和set是最常用的

##### include

这个预处理命令允许我们将配置分解成多个文件。包含指令指定了附加文件的路径，预处理程序将用指定文件的内容替换包含指令。路径可以是单个文件，也可以是通配符，通过使用通配符可以把一个目录中的所有文件都拉进来。在默认配置中，主配置文件(freeswitch.xml)只是一组include指令，其中一个是vars.xml(很快会有更多的内容)，另一个是配置部分的一个。这样，我们就可以为各种系统配置建立一个干净的目录层次结构。例子:

```xml
<!--This include is used by the default configuration to load the vars.xml file into the main configuration file-->
<X-PRE-PROCESS cmd="include" data="vars.xml"/>
<!-- 
    This include directive is used by the default configuration to load all the module configuration files from the autoload_configs directory, this allows us to use a separate configuration file for each module.
 -->
<section name="configuration" description="Various Configuration">
    <X-PRE-PROCESS cmd="include" data="autoload_configs/*.xml"/>
</section>
```

在include文件中,预处理将只包含包含在Include标签中的xml。就像这个例子中看到的那样

```xml
<include>
    <!--content to be included goes here-->
</include>
```

##### set

通过set指令,我们可以设置一个带有值的变量名,然后我们可以在整个配置文件中使用语法\$${variable_name}引用整个变量。当预处理程序运行时,它将使用指定的名称和值创建一个全局变量.全局变量由FreeSWITCH引擎存储,可以在confguration中的任何地方以及从API或者脚本中动态检索.除了设置全局变量,预处理程序还将对配置文件\$\${variable_name}语法的所有文本进行静态替换.在预处理程序运行时,这些文本将被替换成变量的值.由于它是静态替换，所以在使用该变量之前必须先放置set指令.另外,如果在允许时改变了全局变量的值,配置扔将保留旧值.

对于配置中许多地方重复的值,如ip地址和域等不会经常改变的值,set指令很有用.

通过结合set和Include指令,我们可以创建一个单一的文件,所有的变量都被设置了,现在我们可以在许多服务器上共享同一套配置文件,只需要修改这个文件就可以包含服务器的特定设置.事实上,默认配置就是这样配置的,有一个vars.xml的文件,其中设置了ip地址和域名等值.

```xml
<!--Use the Set directive to set the valuable of a global variable-->
<X-PRE-PROCESS cmd="set" data="domain=example.com"/>
  
<!--Use the $${variable_name} syntax to retrieve the value of the variable-->
<action application="bridge" data="sofia/$${domain}/1234@example.com"/>
  
<!-- This is how the previous line will actually appear in the processed configuration file (freeswitch.xml.fsxml) -->
<!-- Note that the reference to the variable is gone -->
<action application="bridge" data="sofia/example.com/1234@example.com"/>
```

##### exec

如果你有一些非常复杂的配置规则，你可以有一个脚本来生成部分配置文件。exec指令将运行一个shell命令，并在配置文件中导入output输出流.

```xml
<X-PRE-PROCESS cmd="exec" data="/path/to/my_script_that_dumps_all_configs_to_stdout.pl"/>
```

##### exec-set

这与exec类似，但它不会将输出流包含在配置文件中，而是设置为一个全局变量

```xml
<!-- Set local_ip_v4 to eth1 address -->
<X-PRE-PROCESS cmd="exec-set" data="local_ip_v4=ip addr show eth1 | awk '/inet /{print $2}' | head -n 1 | cut -d '/' -f 1"/>
```

#### Comment

如果不希望注释出现在最终的XML文件中，可以使用预处理注释.

```xml
<X-PRE-PROCESS cmd="comment" data="This text will be removed by the pre-processor" />
```



