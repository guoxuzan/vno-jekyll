---
layout: post
title: Objective-C类属性
---
**Objective-C从Xcode8起开始支持类属性了。**
>Objective-C now supports class properties, which interoperate with Swift type properties. They are declared as: @property (class) NSString *someStringProperty;. They are never synthesized.

**类属性需要添加一个class参数**
```
@interface Dog : NSObject

@property (class,nonatomic,copy) NSString *name;

@end
```

**类属性不会默认实现set和get方法，需要自己手动实现**

![]({{ site.url }}/assets/2016121601.png)

**实例属性会默认生成对应的成员变量**

![]({{ site.url }}/assets/2016121602.png)

**而类属性不会，需手动添加static变量来实现**

```
#import "Dog.h"

@implementation Dog

static NSString *_name;

+ (void)setName:(NSString *)name {
_name = [name copy];
}

+ (NSString *)name {
return _name;
}

@end
```

**OK，可以正常使用类属性了！**

```
Dog.name = @"二哈";
NSLog(@"%@",Dog.name);
```

![]({{ site.url }}/assets/2016121605.png)



### PS:runtime中API早已区分实例属性和类属性
**(BOOL isInstanceProperty)**
```
/** 
* Returns the specified property of a given protocol.
* 
* @param proto A protocol.
* @param name The name of a property.
* @param isRequiredProperty \c YES searches for a required property, \c NO searches for an optional property.
* @param isInstanceProperty \c YES searches for an instance property, \c NO searches for a class property.
* 
* @return The property specified by \e name, \e isRequiredProperty, and \e isInstanceProperty for \e proto,
*  or \c NULL if none of \e proto's properties meets the specification.
*/
OBJC_EXPORT objc_property_t protocol_getProperty(Protocol *proto, const char *name, BOOL isRequiredProperty, BOOL isInstanceProperty)
OBJC_AVAILABLE(10.5, 2.0, 9.0, 1.0);
```
**从源码可以窥见此API关于类属性的变化** 

**最新版本:**
```
objc_property_t 
protocol_getProperty(Protocol *p, const char *name, 
BOOL isRequiredProperty, BOOL isInstanceProperty)
{
old_protocol *proto = oldprotocol(p);
old_protocol_ext *ext;
old_protocol_list *proto_list;

if (!proto  ||  !name) return nil;

if (!isRequiredProperty) {
// Only required properties are currently supported
return nil;
}

if ((ext = ext_for_protocol(proto))) {
old_property_list *plist;
if (isInstanceProperty) plist = ext->instance_properties;
else if (ext->hasClassPropertiesField()) plist = ext->class_properties;
else plist = nil;

if (plist) {
uint32_t i;
for (i = 0; i < plist->count; i++) {
old_property *prop = property_list_nth(plist, i);
if (0 == strcmp(name, prop->name)) {
return (objc_property_t)prop;
}
}
}
}

if ((proto_list = proto->protocol_list)) {
int i;
for (i = 0; i < proto_list->count; i++) {
objc_property_t prop = 
protocol_getProperty((Protocol *)proto_list->list[i], name, 
isRequiredProperty, isInstanceProperty);
if (prop) return prop;
}
}

return nil;
}
```
**早期版本:**
```
objc_property_t protocol_getProperty(Protocol *p, const char *name, 
BOOL isRequiredProperty, BOOL isInstanceProperty)
{
old_protocol *proto = oldprotocol(p);
old_protocol_ext *ext;
old_protocol_list *proto_list;

if (!proto  ||  !name) return nil;

if (!isRequiredProperty  ||  !isInstanceProperty) {
// Only required instance properties are currently supported
return nil;
}

if ((ext = ext_for_protocol(proto))) {
old_property_list *plist;
if ((plist = ext->instance_properties)) {
uint32_t i;
for (i = 0; i < plist->count; i++) {
old_property *prop = property_list_nth(plist, i);
if (0 == strcmp(name, prop->name)) {
return (objc_property_t)prop;
}
}
}
}

if ((proto_list = proto->protocol_list)) {
int i;
for (i = 0; i < proto_list->count; i++) {
objc_property_t prop = 
protocol_getProperty((Protocol *)proto_list->list[i], name, 
isRequiredProperty, isInstanceProperty);
if (prop) return prop;
}
}

return nil;
}
```
