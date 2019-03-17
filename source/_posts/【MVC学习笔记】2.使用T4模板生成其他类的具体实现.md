---
title: 【MVC学习笔记】2.使用T4模板生成其他类的具体实现
tags:
  - MVC
abbrlink: 18563
date: 2016-09-16 18:35:07
categories:
  - 学习笔记
---
在前篇中我们已经将User类中的代码做了具体的实现，但仍然有多个实体类未实现，以后可能还会增加新的数据表，数据表结构也有可能发生变化，所以我们使用T4模板来完成类的生成，这样就算数据库表发生了改变，也会自动根据改变后的实体对类进行重新生成。
<!-- more -->
### DAL层
下面是数据访问层的T4模板文件 Dal.tt
```csharp
<#@ template language="C#" debug="false" hostspecific="true"#>
<#@ include file="EF.Utility.CS.ttinclude"#><#@
 output extension=".cs"#>
 
<#

CodeGenerationTools code = new CodeGenerationTools(this);
MetadataLoader loader = new MetadataLoader(this);
CodeRegion region = new CodeRegion(this, 1);
MetadataTools ef = new MetadataTools(this);

//EF实体文件在项目中的路径
string inputFile = @"..\\PMS.Model\\PMS.edmx";

EdmItemCollection ItemCollection = loader.CreateEdmItemCollection(inputFile);
string namespaceName = code.VsNamespaceSuggestion();

EntityFrameworkTemplateFileManager fileManager = EntityFrameworkTemplateFileManager.Create(this);

#>
<#//这里为命名空间部分，手动更改为相应的命名空间 #>
using PMS.IDAL;
using PMS.Model;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace PMS.DAL
{
<#
// Emit Entity Types

foreach (EntityType entity in ItemCollection.GetItems<EntityType>().OrderBy(e => e.Name))
{
    //fileManager.StartNewFile(entity.Name + "RepositoryExt.cs");
    //BeginNamespace(namespaceName, code);    
#>        
    public partial class <#=entity.Name#>Dal :BaseDal<<#=entity.Name#>>,I<#=entity.Name#>Dal
    {

    }
<#}#>
    
}
```
我们将EF实体文件路径、命名空间更改为对应的值时，**Ctrl+S** 保存，即可生成对应的其他类型的数据访问类

其他层中也大同小异，只需要做对应的更改即可，下面我将提供相应的代码。
### IDAL层
IDal.tt
```csharp
<#@ template language="C#" debug="false" hostspecific="true"#>
<#@ include file="EF.Utility.CS.ttinclude"#><#@
 output extension=".cs"#> 
<#
CodeGenerationTools code = new CodeGenerationTools(this);
MetadataLoader loader = new MetadataLoader(this);
CodeRegion region = new CodeRegion(this, 1);
MetadataTools ef = new MetadataTools(this);

string inputFile = @"..\\PMS.Model\\PMS.edmx";

EdmItemCollection ItemCollection = loader.CreateEdmItemCollection(inputFile);
string namespaceName = code.VsNamespaceSuggestion();

EntityFrameworkTemplateFileManager fileManager = EntityFrameworkTemplateFileManager.Create(this);

#>
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using PMS.Model;

namespace PMS.IDAL
{
   
<#
// Emit Entity Types

foreach (EntityType entity in ItemCollection.GetItems<EntityType>().OrderBy(e => e.Name))
{
    //fileManager.StartNewFile(entity.Name + "RepositoryExt.cs");
    //BeginNamespace(namespaceName, code);    
#>    
    public partial interface I<#=entity.Name#>Dal :IBaseDal<<#=entity.Name#>>
    {
      
    }
<#}#>
    
}
```
IDbSession.tt
```csharp
<#@ template language="C#" debug="false" hostspecific="true"#>
<#@ include file="EF.Utility.CS.ttinclude"#><#@
 output extension=".cs"#>
 
<#

CodeGenerationTools code = new CodeGenerationTools(this);
MetadataLoader loader = new MetadataLoader(this);
CodeRegion region = new CodeRegion(this, 1);
MetadataTools ef = new MetadataTools(this);

string inputFile = @"..\\PMS.Model\\PMS.edmx";

EdmItemCollection ItemCollection = loader.CreateEdmItemCollection(inputFile);
string namespaceName = code.VsNamespaceSuggestion();

EntityFrameworkTemplateFileManager fileManager = EntityFrameworkTemplateFileManager.Create(this);

#>

using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace PMS.IDAL
{
    public partial interface IDbSession
    {

<#
// Emit Entity Types

foreach (EntityType entity in ItemCollection.GetItems<EntityType>().OrderBy(e => e.Name))
{
    //fileManager.StartNewFile(entity.Name + "RepositoryExt.cs");
    //BeginNamespace(namespaceName, code);    
#>    
        I<#=entity.Name#>Dal <#=entity.Name#>Dal{get;set;}
<#}#>
    }    
}
```
### DALFactory层
SimpleDalFactory.tt
```csharp
<#@ template language="C#" debug="false" hostspecific="true"#>
<#@ include file="EF.Utility.CS.ttinclude"#><#@
 output extension=".cs"#>
 
<#

CodeGenerationTools code = new CodeGenerationTools(this);
MetadataLoader loader = new MetadataLoader(this);
CodeRegion region = new CodeRegion(this, 1);
MetadataTools ef = new MetadataTools(this);

string inputFile =@"..\\PMS.Model\\PMS.edmx";

EdmItemCollection ItemCollection = loader.CreateEdmItemCollection(inputFile);
string namespaceName = code.VsNamespaceSuggestion();

EntityFrameworkTemplateFileManager fileManager = EntityFrameworkTemplateFileManager.Create(this);

#>

using SW.OA.IDAL;
using System;
using System.Collections.Generic;
using System.Configuration;
using System.Linq;
using System.Reflection;
using System.Text;
using System.Threading.Tasks;

namespace SW.OA.DALFactory
{
    public partial class AbstractFactory
    {
      
   
<#
foreach (EntityType entity in ItemCollection.GetItems<EntityType>().OrderBy(e => e.Name))
{    
#>        
        public static I<#=entity.Name#>Dal Create<#=entity.Name#>Dal()
        {

         string fullClassName = NameSpace + ".<#=entity.Name#>Dal";
          return CreateInstance(fullClassName) as I<#=entity.Name#>Dal;

        }
<#}#>
    }
    
}
```
DbSession.tt
```csharp
<#@ template language="C#" debug="false" hostspecific="true"#>
<#@ include file="EF.Utility.CS.ttinclude"#><#@
 output extension=".cs"#>
 
<#

CodeGenerationTools code = new CodeGenerationTools(this);
MetadataLoader loader = new MetadataLoader(this);
CodeRegion region = new CodeRegion(this, 1);
MetadataTools ef = new MetadataTools(this);

string inputFile = @"..\\PMS.Model\\PMS.edmx";

EdmItemCollection ItemCollection = loader.CreateEdmItemCollection(inputFile);
string namespaceName = code.VsNamespaceSuggestion();

EntityFrameworkTemplateFileManager fileManager = EntityFrameworkTemplateFileManager.Create(this);

#>
using PMS.DAL;
using PMS.IDAL;
using PMS.Model;
using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace PMS.DALFactory
{
    public partial class DBSession : IDBSession
    {
<#
// Emit Entity Types

foreach (EntityType entity in ItemCollection.GetItems<EntityType>().OrderBy(e => e.Name))
{
    //fileManager.StartNewFile(entity.Name + "RepositoryExt.cs");
    //BeginNamespace(namespaceName, code);    
#>    
        private I<#=entity.Name#>Dal _<#=entity.Name#>Dal;
        public I<#=entity.Name#>Dal <#=entity.Name#>Dal
        {
            get
            {
                if(_<#=entity.Name#>Dal == null)
                {
                    _<#=entity.Name#>Dal = AbstractFactory.Create<#=entity.Name#>Dal();
                }
                return _<#=entity.Name#>Dal;
            }
            set { _<#=entity.Name#>Dal = value; }
        }
<#}#>
    }    
}
```
### BLL层
Service.tt
```csharp
<#@ template language="C#" debug="false" hostspecific="true"#>
<#@ include file="EF.Utility.CS.ttinclude"#><#@
 output extension=".cs"#>
 
<#

CodeGenerationTools code = new CodeGenerationTools(this);
MetadataLoader loader = new MetadataLoader(this);
CodeRegion region = new CodeRegion(this, 1);
MetadataTools ef = new MetadataTools(this);

string inputFile = @"..\\PMS.Model\\PMS.edmx";

EdmItemCollection ItemCollection = loader.CreateEdmItemCollection(inputFile);
string namespaceName = code.VsNamespaceSuggestion();

EntityFrameworkTemplateFileManager fileManager = EntityFrameworkTemplateFileManager.Create(this);

#>
using PMS.IBLL;
using PMS.Model;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace PMS.BLL
{
<#
// Emit Entity Types

foreach (EntityType entity in ItemCollection.GetItems<EntityType>().OrderBy(e => e.Name))
{
    //fileManager.StartNewFile(entity.Name + "RepositoryExt.cs");
    //BeginNamespace(namespaceName, code);    
#>    
    public partial class <#=entity.Name#>Service :BaseService<<#=entity.Name#>>,I<#=entity.Name#>Service
    {
    

         public override void SetCurrentDal()
        {
            CurrentDal = this.CurrentDbSession.<#=entity.Name#>Dal;
        }
    }   
<#}#>
    
}
```
### IBLL层
IService.tt
```csharp
<#@ template language="C#" debug="false" hostspecific="true"#>
<#@ include file="EF.Utility.CS.ttinclude"#><#@
 output extension=".cs"#>
 
<#

CodeGenerationTools code = new CodeGenerationTools(this);
MetadataLoader loader = new MetadataLoader(this);
CodeRegion region = new CodeRegion(this, 1);
MetadataTools ef = new MetadataTools(this);

string inputFile = @"..\\PMS.Model\\PMS.edmx";

EdmItemCollection ItemCollection = loader.CreateEdmItemCollection(inputFile);
string namespaceName = code.VsNamespaceSuggestion();

EntityFrameworkTemplateFileManager fileManager = EntityFrameworkTemplateFileManager.Create(this);

#>

using PMS.Model;
using PMS.Model.Search;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace PMS.IBLL
{
<#
// Emit Entity Types

foreach (EntityType entity in ItemCollection.GetItems<EntityType>().OrderBy(e => e.Name))
{
    //fileManager.StartNewFile(entity.Name + "RepositoryExt.cs");
    //BeginNamespace(namespaceName, code);    
#>    
    public partial interface I<#=entity.Name#>Service : IBaseService<<#=entity.Name#>>
    {
       
    }   
<#}#>
    
}
```
至此，我们完成了基本框架内容的填充。