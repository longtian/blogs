---
title: Pycharm 里为 Vagrantfile 如何设置语法高亮
date: 2020-07-09 10:00:25
tags:
 - vagrant
 - pycharm
---

最近又开始捣鼓 `Vagrant` 了，但是在 Pycharm (2020.1.2) 里一直没有语法高亮，明明装了 `Vagrant` 插件的。

还好网上搜到了这个[方法](https://gist.github.com/boneskull/efcf2dcf265096f014d7#file-vagrantfile-xml)，新建 `Vagrantfile.xml` 如下

```xml
<filetype binary="false" description="Vagrant Configuration File" name="Vagrantfile">
  <highlighting>
    <options>
      <option name="LINE_COMMENT" value="#" />
      <option name="COMMENT_START" value="=begin" />
      <option name="COMMENT_END" value="=end" />
      <option name="HEX_PREFIX" value="" />
      <option name="NUM_POSTFIXES" value="" />
      <option name="HAS_BRACES" value="true" />
      <option name="HAS_BRACKETS" value="true" />
      <option name="HAS_PARENS" value="true" />
      <option name="HAS_STRING_ESCAPES" value="true" />
    </options>
    <keywords keywords="BEGIN;END;begin;break;case;do;else;elsif;end;ensure;for;if;in;next;rescue;retry;then;until;when;while" ignore_case="false" />
    <keywords2 keywords="__ENCODING__;__END__;__FILE__;__LINE__" />
    <keywords3 keywords="and;false;nil;not;or;true" />
    <keywords4 keywords="class;def;module;return;self;super;undef;yield" />
  </highlighting>
  <extensionMap>
    <mapping pattern="Vagrantfile" />
  </extensionMap>
</filetype>
```

貌似得放到某个文件夹下，懒得找了，直接在 IDE 里设置吧：

进入 ` File > Settings > Editor > File Types ` 对话框，在 `Recognized file types:` 里点击 `+` 号，
添加 `Vagrantfile` 类型即可，其它的配置参照上面 `Vagrantfile.xml` 里，

唯一要注意的是 `Keywords` 部分不是以 `;` 分割而是需要一行一个 `Keyword`

**错误**

```
__ENCODING__;__END__;__FILE__;__LINE__
```

**正确**

```
__ENCODING__
__END__
__FILE__
__LINE__
```

用这种方法还可为任何类型的文件增加语法高亮，是不是很赞！