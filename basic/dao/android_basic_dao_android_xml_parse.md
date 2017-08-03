# Android xml 之DOM、SAX、PULL

案例演示xml文档结构(test.xml)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<persons>
    <person id="1">
        <name>小罗</name>
        <age>21</age>
    </person>
    <person id="2">
        <name>android</name>
        <age>15</age>
    </person>
</persons>
```
## DOM

Android 完全支持DOM 解析。DOM（Document ObjectModel）是文档对象模型。利用DOM 中的对象，可以对XML 文档进行读取、搜索、修改、添加和删除等操作。
使用DOM 对XML 文件进行操作时，首先要解析文件，将文件分为独立的元素、属性和注释等，然后以节点树的形式在内存中对XML文件进行表示，就可以通过节点树访问文档的内容，并根据需要修改文档——这就是DOM的工作原理。DOM实现时首先为XML文档的解析定义一组接口，解析器读入整个文档，然后构造一个驻留内存的树结构，这样代码就可以使用DOM接口来操作整个树结构。
DOM 解析流程

![img](/imgs/android_xml_parse_dom.jpg)
  - dom解析xml时会将整个xml文档添加到文档中，因此可以很快速地定位到需要的标签上，同时如果有需要还可以修改。
  - 但是由于要将整个xml文档加载到内存，这对于移动设备等内存很小的设备来说很危险，所以这种解析常用在服务器开发上。

Java代码

```java
private void xmlWithDOM() {
        InputStream is = null;
        DocumentBuilder documentBuilder = null;
        Document document = null;
        try {
            is = getAssets().open("test.xml");
            //创建一个Document对象
            DocumentBuilderFactory builderFactory = DocumentBuilderFactory.newInstance();
            documentBuilder = builderFactory.newDocumentBuilder();
            //加载被解析的文档
            document = documentBuilder.parse(is);
            //获得xml的根节点
            Element rootElement = document.getDocumentElement();
            //获得标签为person的所有节点
            NodeList nodes = rootElement.getElementsByTagName("person");
            for (int i = 0; i < nodes.getLength(); i++) {
                //判断node的类型是否叶子节点
                if (nodes.item(i).getNodeType() == Document.ELEMENT_NODE) {
                    Element element = (Element) nodes.item(i);
                    String id = element.getAttribute("id");
                    Log.d("person--id:", id);
                    NodeList childNodes = element.getChildNodes();
                    for (int j = 0; j < childNodes.getLength(); j++) {
                        if (childNodes.item(j).getNodeType() == Document.ELEMENT_NODE) {
                            Node node = childNodes.item(j);
                            if ("name".equals(node.getNodeName())) {
                                Log.d("person--name:", node.getFirstChild().getNodeValue());
                            } else if ("age".equals(node.getNodeName())) {
                                Log.d("person--age:", node.getFirstChild().getNodeValue());
                            }
                        }

                    }
                }

            }
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ParserConfigurationException pe) {
            pe.printStackTrace();
        } catch (SAXException se) {
            se.printStackTrace();
        } finally {
            is = null;
            document = null;
        }
    }
```
## SAX

SAX（Simple API for XML）XML简单应用程序接口是一个公共的基于事件的XML文档解析标准。它以事件作为解析XML文件的模式，它将XML文件转化成一系列的事件，由不同的事件处理器来决定如何处理。SAX是一个解析速度快并且占用内存少的xml解析器，非常适合android等移动设备，SAX解析XML文件采用的是事件驱动，也就是说，它并不需要解析完整个文档，在按内容顺序解析文档的过程中，SAX会判断当前读取到的字符是否合法xml语法中的某部分，如果符合就会触发事件。
SAX解析流程
![img](/imgs/android_xml_parse_sax.jpg)
  - SAX是一个解析速度快并且占用内存少的xml解析器。
  - SAX解析XML文件采用的是事件驱动,当解析到相应地标签时就会触发相应地事件。

回调Handler：
```java
class SAXHandler extends DefaultHandler {
    String elementTag;

    @Override
    public void startDocument() throws SAXException {
        Log.d("SAX---parsexml:", "startDocument");
        super.startDocument();
    }

    @Override
    public void startElement(String uri, String localName, String qName, Attributes attributes) throws SAXException {
        Log.d("SAX---parsexml:", "startElement");
        elementTag = qName;

        super.startElement(uri, localName, qName, attributes);
    }


    @Override
    public void characters(char[] ch, int start, int length) throws SAXException {
        Log.d("SAX---parsexml:", "characters");
        String value = new String(ch, start, length);
        if ("name".equals(elementTag)) {
            Log.d("person--name:", value);
        } else if ("age".equals(elementTag)) {
            Log.d("person--age:", value);
        }
        elementTag = null;
        super.characters(ch, start, length);
    }

    @Override
    public void endElement(String uri, String localName, String qName) throws SAXException {
        Log.d("SAX---parsexml:", "endElement");
        super.endElement(uri, localName, qName);
    }

    @Override
    public void endDocument() throws SAXException {
        Log.d("SAX---parsexml:", "endDocument");
        super.endDocument();
    }
}
```
使用代码:
```java
private void xmlWithSAX() {
        try {
            InputStream is = getAssets().open("test.xml");
            SAXParserFactory saxParserFactory = SAXParserFactory.newInstance();
            SAXParser saxParser = saxParserFactory.newSAXParser();
            saxParser.parse(is, new SAXHandler());

        } catch (IOException io) {
            io.printStackTrace();
        } catch (SAXException se) {
            se.printStackTrace();
        } catch (ParserConfigurationException pe) {
            pe.printStackTrace();
        }


    }
```
## PULL

在Android API 中，另外提供了Android.util.Xml类，同样可以解析XML文件，使用方法类似SAX，也都需编写Handler来处理XML的解析，但是在使用上却比SAX来得简单。它允许用户的应用程序代码从解析器中获取事件，这与SAX解析器自动将事件推入处理程序相反。Pull解析器运行方式与SAX解析器类似，它提供了类似ide事件，如：开始元素和结束元素，使用parser.next()可以进入下一个元素并触发相应的事件。事件作为数值代码被发送，因此可以使用一个switch对感兴趣的事件进行处理。当元素开始解析时，调用parser.nextText()方法获取一个Text类型的节点的值。

PULL解析流程
![img](/imgs/android_xml_parse_pull.jpg)
  - pull解析也是事件驱动型的xml解析，这一点和sax解析类似，不过pull需要调用next() 方法提取它们（主动提取事件）。
  - 而SAX解析器的工作方式是自动将事件推入注册的事件处理器进行处理，因此你不能控制事件的处理主动结束；
  - 因此pull解析除了具备sax优点外程序控制上更加灵活。

JAVA代码：
```java
private void xmlPULL() {
        try {
            //加载文件
            InputStream is = getAssets().open("test.xml");
            //构造pull解析对象
            XmlPullParserFactory xmlPullParserFactory = XmlPullParserFactory.newInstance();
            XmlPullParser xmlPullParser = xmlPullParserFactory.newPullParser();
            //加载流
            xmlPullParser.setInput(is, "utf-8");
            //获取事件类型
            int eventType = xmlPullParser.getEventType();

            while (eventType != XmlPullParser.END_DOCUMENT) {
                switch (eventType) {
                    case XmlPullParser.START_DOCUMENT:
                        Log.d("PULL", "Start Document");
                        break;
                    case XmlPullParser.START_TAG:
                        if ("person".equals(xmlPullParser.getName())) {
                            Log.d("id", xmlPullParser.getAttributeValue(0));
                        } else if ("name".equals(xmlPullParser.getName())) {
                            Log.d("name", xmlPullParser.nextText());
                        } else if ("age".equals(xmlPullParser.getName())) {
                            Log.d("age", xmlPullParser.nextText());
                        }
                        break;
                    case XmlPullParser.END_DOCUMENT:
                        Log.d("PULL", "End Document");
                        break;
                    default:
                        break;
                }
                eventType = xmlPullParser.next();
            }


        } catch (IOException io) {
            io.printStackTrace();
        } catch (XmlPullParserException xfe) {
            xfe.printStackTrace();
        }

    }
```