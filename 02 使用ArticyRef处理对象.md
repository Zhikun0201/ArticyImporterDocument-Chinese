# 02 使用ArticyRef处理对象 Object Handling with ArticyRef

在此我们将向您展示如何在自定义Unity脚本中为articy对象设置参考（references），以便在Unity检查窗口中修改，或是在C#脚本中获取。

Here we show you how you can set references to articy objects in your custom unity scripts to modify them in the Unity inspector and how to access them inside your C# scripts.

本篇包含以下内容：

This topic contains the following sections:

- 开始使用ArticyRef [Getting Started with ArticyRef](https://www.articy.com/articy-importer/unity/html/howto_articyref.htm#SimpleAccessByRef)
- ArticyRef的类型约束 [Type constraints for ArticyRef](https://www.articy.com/articy-importer/unity/html/howto_articyref.htm#TypeConstraint)
- ArticyRef的进阶使用 [Advanced uses of ArticyRef](https://www.articy.com/articy-importer/unity/html/howto_articyref.htm#AdvancedArticyRef)
- 相关连接 [See Also](https://www.articy.com/articy-importer/unity/html/howto_articyref.htm#seeAlsoSection)

## ArticyRef的使用 Getting Started with ArticyRef

与其将Articy对象的技术名称硬编码到脚本中，不如让引用（references）来保持代码的通用性和可重用性。你可以像其他任何公共变量一样，在Unity Inspector中设置对Articy对象的引用：

Rather than hardcoding the Technical Name of your Articy objects into your scripts, you can keep your code generic and reusable by working with references instead. You can set references to Articy objects in the Unity Inspector just like any other public variable:

请创建新的C#脚本，命名为MyArticyScript，并写入如下代码：

Create a new C# script file called MyArticyScript and paste the following code:

```
using UnityEngine;
using System.Collections;
using Articy.Unity; 
public class MyArticyScript : MonoBehaviour 
{    
    public ArticyRef myRef;     

    // Use this for initialization    
    void Start () {     }     

    // Update is called once per frame    
    void Update () {     } 
}
```

请注意公开声明ArticyRef类型的变量myRef，并且不要忘记为ArticyRef添加名称空间Articy.Unity。

Note the public field myRef of type [ArticyRef](https://www.articy.com/articy-importer/unity/html/T_Articy_Unity_ArticyRef.htm) and don't forget to add the namespace for the [ArticyRef](https://www.articy.com/articy-importer/unity/html/T_Articy_Unity_ArticyRef.htm) using Articy.Unity.



保存文件并回到Unity当中，将看到Inspector中显示的变量。

Saving the file and going back to unity will show the exposed variable in the Inspector:

![empty ref](https://pic1.cdncl.net/user/user_upload_osl/senyi/d422046a28c875a3aa2940c66278b3de.jpg)

现在可以从articy数据库中直接选择对象，只需点击右侧的浏览按钮，选择需要参考的对象。

Now you can select your object directly from the articy database by clicking the browse button on the right side of the textbox and using the object picker to select the reference

![ref object picker manfred](https://pic1.cdncl.net/user/user_upload_osl/senyi/751f9fd704affbf1e4fd11c03e296c82.jpg)

双击目标对象

and double clicking the desired object

![manfred ref](https://pic1.cdncl.net/user/user_upload_osl/senyi/94523f018ebd8db1f5948f5b29e7b116.jpg)

将在区域中存储字段，以便在代码中使用。

will store the reference in our field and its ready to be used in code.



现在可以开始访问引用对象。以下代码将验证引用是否有效，并将其技术名称打印到控制台：

Let's see how we can access the referenced object now. The following code will verify that the reference is valid - and then print out its Technical Name to the console:

```
using UnityEngine;
using System.Collections;
using Articy.Unity; 
public class MyArticyScript : MonoBehaviour 
{    
    public ArticyRef myRef;     
    
    // Use this for initialization    
    void Start ()     
    {        
        if(myRef.HasReference)        
        {            
               var obj = myRef.GetObject();            
            Debug.Log(obj.TechnicalName);        
          }    
    }     

    // Update is called once per frame    
    void Update () 
    {     
    } 
}
```

请注意如何检查myRef，在使用HasReference获取它之前，要保证该参考已经在Unity当中设置完成。对null进行简单检查是不够的，因为ArtifyRef仅仅是一个打包的类。为了获取准确对象，需要调用GetObject()。返回的值类型为ArticyObject，可以将其转换为任何对象或模板类型，详情可参阅“基本对象处理”部分。

Notice how we check our myRef before we access it using [HasReference](https://www.articy.com/articy-importer/unity/html/P_Articy_Unity_ArticyRef_HasReference.htm) making sure that the reference was set in Unity. A simple check against null is not sufficient, because ArtifyRef is merely a wrapper class. To get the actual object, we need to call GetObject(). The returned object is of type [ArticyObject](https://www.articy.com/articy-importer/unity/html/T_Articy_Unity_ArticyObject.htm), which can be cast into any object or template type just as demonstrated in the section [Basic Object handling](https://www.articy.com/articy-importer/unity/html/howto_objects.htm).

```
if(myRef.HasReference) 
{    
	var obj = myRef.GetObject<Player_Character>(); 
	// 由于指定了更具体的类型，现在可以访问角色属性了
	Debug.Log(obj.DisplayName); 
}
```

请注意如何将所需的类型传递到GetObject方法中。就像用GameObject.GetComponent一样。

Notice how we pass in the desired type into our GetObject method, just like we do with [GameObject.GetComponent](https://docs.unity3d.com/ScriptReference/GameObject.GetComponent.html).



## ArticyRef的类型约束 Type constraints for ArticyRef

默认情况下，ArticyRef接受articy对象的所有类型。这可能会造成使用错误。举例来说，代码在是Character类型的情况下，我们仍有可能会在Inspector中将ArticyRef设置为FlowFragment类型的数据。当脚本运行时就可能会遭遇NullReferenceException错误。为免发生此类错误，可限制ArticyRef的对象选择器显示的内容。也即类型限制。

By default, ArticyRef accepts all types of articy objects. This can lead to user error. The code might for example be expecting an object of type Character, but nobody is stopping us from setting a FlowFragment in our ArticyRef in the Inspector. We could run into a potential NullReferenceException when our script runs. To remove this potential trap, we can limit what the object picker for our ArticyRef will display. This is called a Type constraint

下方代码将会展示如何限制对象为Character

The code below demonstrates how to limit the objects that can be assigned to our exposed variable to objects of the type Character.

```
[ArticyTypeConstraint(typeof(Character))]
public ArticyRef playerProtagonist;
```

现在当我们点击Inspector的浏览按钮时，就只会看见Character类型的对象了

Now when we click the browse button in the inspector, we only see objects that are of type Character

![ref object character constraint](https://pic1.cdncl.net/user/user_upload_osl/senyi/89b3b5030fcd6f9f02f401c8f23dd4ca.jpg)

| ![Note](https://pic1.cdncl.net/user/user_upload_osl/senyi/d821fa7345bbacb96bdc0e7ec0736ce5.jpg) 注意 |
| ------------------------------------------------------------ |
| 请注意，类型约束只是个视觉层面的过滤器，用于在Unity界面引导使用，并不会更改ArticyRef内部的工作方式。仍可通过脚本分配任何类型的Articy对象。 |



## ArticyRef的进阶使用 Advanced uses of ArticyRef

使用对象接口（如typeof(IEntity)）和基本类型（如typeof(Entity)）存在重要区别：

There is an important difference in using an object interface, like typeof(IEntity), and a base type, like typeof(Entity)

- *Interface*: 显示所有带有或未带有模板的对象，如typepf(IEntity)将会列出所有类Entity类型的对象，如角色、道具等。Shows all Objects with or without template. eg. typeof(IEntity) would list objects of type Entity, Character, Item etc.
- *Base Type*: 仅显示不带模板的类。如typeof(Entity)将会列出不带模板的Entity。Show **only** classes without templates. eg. typeof(Entity) Would only list Entities without a template.

这似乎有点不直观，但我们可以简单展示最常用的用例。

This may seem a bit unintuitive, but this way we can easily represent the most frequent use cases.



你可以添加任何数量的限制。但请注意，为了选择器中能有确定的对象，至少满足一个约束条件

You can combine as many type constraints as you want. But remember, to have a specific object shown in the picker at least **one** of the constraints must match

```
[ArticyTypeConstraint(typeof(Character), typeof(Item))]
public ArticyRef requiredPossession;
```

利用对象和属性接口，我们可以获取到非常精准的类型

Utilizing object and property interfaces, we can ask for very specific types

```
// 仅实体，无论是否有模板
[ArticyTypeConstraint(typeof(IEntity))]
public ArticyRef allEntities; 

// 仅实体，没有模板
 [ArticyTypeConstraint(typeof(Entity))]
public ArticyRef onlyEntities; 

// 只要带有DisplayName属性就可
 [ArticyTypeConstraint(typeof(IObjectWithDisplayName))]
public ArticyRef anythingWithDisplayName; 

// 任何带有模板的对象即可，只要含有“SoundFile”属性即可
 [ArticyTypeConstraint(typeof(IObjectWithFeatureSoundfile))]
public ArticyRef anyTemplatedObjectWithFeatureSoundFile; 
```

也可以在需要时使用列表

If you want you can use lists of articy refs

```
// 你可以使用ArticyRef列表
public List<ArticyRef> manyRefs; 
```

并对列表中的每个元素使用单一类型约束

And use a single type constraint for every element in the list

```
// 并可限制他们
[ArticyTypeConstraint(typeof(Character))]
public List<ArticyRef> selectableCharacters; 
```

## ![baa154eb6fbcb9b871d614e0c9dfdce9.jpg](https://pic1.cdncl.net/user/user_upload_osl/senyi/baa154eb6fbcb9b871d614e0c9dfdce9.jpg)相关链接 See Also

#### Reference

[ArticyRef](https://www.articy.com/articy-importer/unity/html/T_Articy_Unity_ArticyRef.htm)

[GetObject()](https://www.articy.com/articy-importer/unity/html/M_Articy_Unity_ArticyRef_GetObject.htm)

[ArticyTypeConstraintAttribute](https://www.articy.com/articy-importer/unity/html/T_Articy_Unity_ArticyTypeConstraintAttribute.htm)

[ArticyDatabase](https://www.articy.com/articy-importer/unity/html/T_Articy_Unity_ArticyDatabase.htm)

#### Other Resources

[Basic Object handling](https://www.articy.com/articy-importer/unity/html/howto_objects.htm)

[Object Templates](https://www.articy.com/articy-importer/unity/html/howto_templates.htm)

[Cloning Objects](https://www.articy.com/articy-importer/unity/html/howto_clone.htm)

[(Unity Doc) GameObject.GetComponent](https://docs.unity3d.com/ScriptReference/GameObject.GetComponent.html)