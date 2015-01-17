## CocoaMarkdown
### Markdown parsing and rendering for iOS and OS X

CocoaMarkdown is a cross-platform framework for parsing and rendering Markdown, built on top of the [C reference implementation](https://github.com/jgm/CommonMark) of [CommonMark](http://commonmark.org).

**This is currently beta-quality code.**

### Why?

CocoaMarkdown aims to solve two primary problems better than existing libraries:

1. **More flexibility**. CocoaMarkdown allows you to define custom parsing hooks or even traverse the Markdown AST using the low-level API.
2. **Efficient `NSAttributedString` creation for easy rendering on iOS and OS X**. Most existing libraries just generate HTML from the Markdown, which is not a convenient representation to work with in native apps.

### API

#### Traversing the Markdown AST

`CMNode` and `CMIterator` wrap CommonMark's C types with an object-oriented interface for traversal of the Markdown AST.

```
let document = CMDocument(contentsOfFile: path)
document.rootNode.iterator().enumerateUsingBlock { (node, _, _) in
    print("String value: \(node.stringValue)")
}
```

#### Building Custom Renderers

The `CMParser` class isn't _really_ a parser (it just traverses the AST), but it defines an `NSXMLParser`-style delegate API that provides handy callbacks for building your own renderers:

```
@protocol CMParserDelegate <NSObject>
@optional
- (void)parserDidStartDocument:(CMParser *)parser;
- (void)parserDidEndDocument:(CMParser *)parser;
...
- (void)parser:(CMParser *)parser foundText:(NSString *)text;
- (void)parserFoundHRule:(CMParser *)parser;
...
@end
```

`CMAttributedStringRenderer` is an example of a custom renderer that is built using this API.

#### Generating Attributed Strings

`CMAttributedStringRenderer` is the high level API that will be useful to most apps. It creates an `NSAttributedString` directly from Markdown, skipping the step of converting it to HTML altogether.

Going from a Markdown document to rendering it on screen is as easy as:

```
let document = CMDocument(contentsOfFile: path)
let renderer = CMAttributedStringRenderer(document: document, attributes: CMTextAttributes())
textView.attributedText = renderer.render()
```

All attributes used to style the text are customizable using the `CMTextAttributes` class:

```
let attributes = CMTextAttributes()
attributes.linkAttributes = [
    NSForegroundColorAttributeName: UIColor.redColor()
]
attributes.emphasisAttributes = [
    NSBackgroundColorAttributeName: UIColor.yellowColor()
]
```

HTML elements can be supported by implementing `CMHTMLElementTransformer`. The framework includes several transformers for commonly used tags:

* `CMHTMLStrikethroughTransformer`
* `CMHTMLSuperscriptTransformer`
* `CMHTMLSubscriptTransformer`

Transformers can be registered with the renderer to use them:

```
...
renderer.registerHTMLElementTransformer(CMHTMLStrikethroughTransformer())       renderer.registerHTMLElementTransformer(CMHTMLSuperscriptTransformer())
textView.attributedText = renderer.render()
```

### Example Apps

The project includes example apps for iOS and OS X to demonstrate rendering attributed strings.

### Contact

* Indragie Karunaratne
* [@indragie](http://twitter.com/indragie)
* [http://indragie.com](http://indragie.com)

### License

CocoaMarkdown is licensed under the MIT License. See `LICENSE` for more information.