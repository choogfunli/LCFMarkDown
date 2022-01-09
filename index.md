## iOS多线程编程

You can use the [editor on GitHub](https://github.com/choogfunli/LCFMarkDown/edit/gh-pages/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### NSThread

NSThread使用

1、实例初始化、

```markdown
Syntax highlighted code block

 //创建线程
 NSThread *newThread = [[NSThread alloc]initWithTarget:self selector:@selector(demo:) object:@"Thread"];
 //或者
 NSThread  *newThread=[[NSThread alloc]init];
 NSThread  *newThread= [[NSThread alloc]initWithBlock:^{
     NSLog(@"initWithBlock");
 }];

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [Basic writing and formatting syntax](https://docs.github.com/en/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax).

### GCD

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/choogfunli/LCFMarkDown/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### NSOperation

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
