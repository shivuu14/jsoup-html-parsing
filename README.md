# jsoup を使った HTML のパース

[![Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.jp/) 

このガイドでは、Java で `jsoup` を使って HTML をパースする方法を説明します。DOM メソッドの使い方、ページネーションの処理、そしてパースのワークフローを最適化する方法を学べます。

- [Jsoup で DOM メソッドを使う](#using-dom-methods-with-jsoup)
  - [getElementById](#getelementbyid)
  - [getElementsByTag](#getelementsbytag)
  - [getElementsByClass](#getelementsbyclass)
  - [getElementsByAttribute](#getelementsbyattribute)
- [高度なテクニック](#advanced-techniques)
  - [CSS セレクター](#css-selectors)
  - [ページネーションの処理](#handling-pagination)
- [すべてをまとめる](#putting-everything-together)

## Getting Started

このチュートリアルでは、依存関係の管理に [Maven](https://maven.apache.org/) を使用することを前提とします。

Maven をインストールしたら、`jsoup-scraper` という新しい Java プロジェクトを作成します。

```bash
mvn archetype:generate -DgroupId=com.example -DartifactId=jsoup-scraper -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

関連する依存関係を追加するには、`pom.xml` のコードを以下のコードに置き換えます。

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>jsoup-scraper</artifactId>
  <packaging>jar</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>jsoup-scraper</name>
  <url>http://maven.apache.org</url>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.jsoup</groupId>
        <artifactId>jsoup</artifactId>
        <version>1.16.1</version>
    </dependency>
  </dependencies>
  <properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
</properties>
</project>
```

次に、以下のコードを `App.java` に貼り付けます。

```java
package com.example;

import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;
import org.jsoup.select.Elements;

public class App {
    public static void main(String[] args) {

        String url = "https://books.toscrape.com";
        int pageCount = 1;

        while (pageCount <= 1) {

            try {
                System.out.println("---------------------PAGE "+pageCount+"--------------------------");

                //connect to a website and get its HTML
                Document doc = Jsoup.connect(url).get();
            
                //print the title
                System.out.println("Page Title: " + doc.title());
            
                
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        System.out.println("Total pages scraped: "+(pageCount-1));
    }
}
```

- `Jsoup.connect("https://books.toscrape.com").get()`: この行はページを取得し、操作可能な `Document` オブジェクトを返します。
- `doc.title()` は HTML ドキュメント内の title を返します。この場合は `All products | Books to Scrape - Sandbox` です。

## Jsoup で DOM メソッドを使う

`jsoup` には、DOM（Document Object Model）内の要素を見つけるためのさまざまなメソッドが含まれています。以下のいずれかを使って、ページ要素を簡単に見つけられます。

- `getElementById()`: `id` を使って要素を見つけます。
- `getElementsByClass()`: CSS クラスを使ってすべての要素を見つけます。
- `getElementsByTag()`: HTML タグを使ってすべての要素を見つけます。
- `getElementsByAttribute()`: 特定の属性を含むすべての要素を見つけます。

### getElementById

スクレイピング対象の Web サイトでは、サイドバーに `id` が `promotions_left` の `div` があります。

![Inspect the sidebar](https://github.com/luminati-io/jsoup-html-parsing/blob/main/Images/Inspect-the-sidebar.png)

```java
//get by Id
Element sidebar = doc.getElementById("promotions_left");

System.out.println("Sidebar: " + sidebar);
```

このコードは、Inspect 画面で見える HTML 要素を出力します。

```
Sidebar: <div id="promotions_left">
</div>
```

### getElementsByTag

`getElementsByTag()` を使うと、特定のタグを持つページ内のすべての要素を見つけられます。このページでは、各書籍が固有の `article` タグに含まれています。

![Inspect books](https://github.com/luminati-io/jsoup-html-parsing/blob/main/Images/Inspect-books.png)

以下のコードは書籍の配列を返し、以降のデータ取得の基礎になります。

```java
//get by tag
Elements books = doc.getElementsByTag("article");
```

### getElementsByClass

書籍の価格を確認してみましょう。クラスは `price_color` です。

![Inspect price](https://github.com/luminati-io/jsoup-html-parsing/blob/main/Images/Inspect-price.png)

以下のコードスニペットは、`price_color` クラスのすべての要素を見つけ、`.first().text()` を使って最初の要素のテキストを出力します。

```java
System.out.println("Price: " + book.getElementsByClass("price_color").first().text());
```

### getElementsByAttribute

`getElementsByAttribute("href")` を使って、`href` 属性を持つすべての要素を見つけましょう。

```java
//get by attribute
Elements hrefs = book.getElementsByAttribute("href");
System.out.println("Link: https://books.toscrape.com/" + hrefs.first().attr("href"));
```

## 高度なテクニック

### CSS セレクター

複数条件で要素を見つけるには、`select()` メソッドに CSS セレクターを渡します。これにより、セレクターに一致するすべてのオブジェクトの配列が返されます。次のコードスニペットでは、`li[class='next']` を使って `next` クラスを持つすべての `li` アイテムを見つけます。

```java
Elements nextPage = doc.select("li[class='next']");
```

### ページネーションの処理

ページネーションを処理するには、まず `nextPage.first()` を使って配列から最初の要素を取得します。次に、その要素に対して `getElementsByAttribute("href").attr("href")` を呼び出し、`href` の値を抽出します。

2 ページ目以降ではリンクから `catalogue` という単語が削除されるため、`catalogue` を含まない場合は `href` を追加して戻します。その後、この更新されたリンクをベース URL と結合して、次ページの URL を取得します。

```java
if (!nextPage.isEmpty()) {
    String nextUrl = nextPage.first().getElementsByAttribute("href").attr("href");
    if (!nextUrl.contains("catalogue")) {
        nextUrl = "catalogue/"+nextUrl;
    } 
    url = "https://books.toscrape.com/" + nextUrl;
    pageCount++;
}
```

## すべてをまとめる

以下が最終的な Java コードです。複数ページをスクレイピングするには、`while (pageCount <= 1)` の `1` を変更するだけです。例えば 4 ページをスクレイピングする場合は、`while (pageCount <= 4)` を使用します。

```java
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;
import org.jsoup.select.Elements;

public class App {
    public static void main(String[] args) {

        String url = "https://books.toscrape.com";
        int pageCount = 1;

        while (pageCount <= 1) {

            try {
                System.out.println("---------------------PAGE "+pageCount+"--------------------------");

                //connect to a website and get its HTML
                Document doc = Jsoup.connect(url).get();
            
                //print the title
                System.out.println("Page Title: " + doc.title());
            
                //get by Id
                Element sidebar = doc.getElementById("promotions_left");

                System.out.println("Sidebar: " + sidebar);

                //get by tag
                Elements books = doc.getElementsByTag("article");

                for (Element book : books) {
                    System.out.println("------Book------");
                    System.out.println("Title: " + book.getElementsByTag("img").first().attr("alt"));
                    System.out.println("Price: " + book.getElementsByClass("price_color").first().text());
                    System.out.println("Availability: " + book.getElementsByClass("instock availability").first().text());

                    //get by attribute
                    Elements hrefs = book.getElementsByAttribute("href");
                    System.out.println("Link: https://books.toscrape.com/" + hrefs.first().attr("href"));
                }

                //find the next button using its CSS selector
                Elements nextPage = doc.select("li[class='next']");
                if (!nextPage.isEmpty()) {
                    String nextUrl = nextPage.first().getElementsByAttribute("href").attr("href");
                    if (!nextUrl.contains("catalogue")) {
                        nextUrl = "catalogue/"+nextUrl;
                    } 
                    url = "https://books.toscrape.com/" + nextUrl;
                    pageCount++;
                }

            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        System.out.println("Total pages scraped: "+(pageCount-1));
    }
}
```

コードをコンパイルします。

```bash
mvn package
```

次に実行できます。

```bash
mvn exec:java -Dexec.mainClass="com.example.App"
```

以下は 1 ページ目の出力です。

```
---------------------PAGE 1--------------------------
Page Title: All products | Books to Scrape - Sandbox
Sidebar: <div id="promotions_left">
</div>
------Book------
Title: A Light in the Attic
Price: £51.77
Availability: In stock
Link: https://books.toscrape.com/catalogue/a-light-in-the-attic_1000/index.html
------Book------
Title: Tipping the Velvet
Price: £53.74
Availability: In stock
Link: https://books.toscrape.com/catalogue/tipping-the-velvet_999/index.html
------Book------
Title: Soumission
Price: £50.10
Availability: In stock
Link: https://books.toscrape.com/catalogue/soumission_998/index.html
------Book------
Title: Sharp Objects
Price: £47.82
Availability: In stock
Link: https://books.toscrape.com/catalogue/sharp-objects_997/index.html
------Book------
Title: Sapiens: A Brief History of Humankind
Price: £54.23
Availability: In stock
Link: https://books.toscrape.com/catalogue/sapiens-a-brief-history-of-humankind_996/index.html
------Book------
Title: The Requiem Red
Price: £22.65
Availability: In stock
Link: https://books.toscrape.com/catalogue/the-requiem-red_995/index.html
------Book------
Title: The Dirty Little Secrets of Getting Your Dream Job
Price: £33.34
Availability: In stock
Link: https://books.toscrape.com/catalogue/the-dirty-little-secrets-of-getting-your-dream-job_994/index.html
------Book------
Title: The Coming Woman: A Novel Based on the Life of the Infamous Feminist, Victoria Woodhull
Price: £17.93
Availability: In stock
Link: https://books.toscrape.com/catalogue/the-coming-woman-a-novel-based-on-the-life-of-the-infamous-feminist-victoria-woodhull_993/index.html
------Book------
Title: The Boys in the Boat: Nine Americans and Their Epic Quest for Gold at the 1936 Berlin Olympics
Price: £22.60
Availability: In stock
Link: https://books.toscrape.com/catalogue/the-boys-in-the-boat-nine-americans-and-their-epic-quest-for-gold-at-the-1936-berlin-olympics_992/index.html
------Book------
Title: The Black Maria
Price: £52.15
Availability: In stock
Link: https://books.toscrape.com/catalogue/the-black-maria_991/index.html
------Book------
Title: Starving Hearts (Triangular Trade Trilogy, #1)
Price: £13.99
Availability: In stock
Link: https://books.toscrape.com/catalogue/starving-hearts-triangular-trade-trilogy-1_990/index.html
------Book------
Title: Shakespeare's Sonnets
Price: £20.66
Availability: In stock
Link: https://books.toscrape.com/catalogue/shakespeares-sonnets_989/index.html
------Book------
Title: Set Me Free
Price: £17.46
Availability: In stock
Link: https://books.toscrape.com/catalogue/set-me-free_988/index.html
------Book------
Title: Scott Pilgrim's Precious Little Life (Scott Pilgrim #1)
Price: £52.29
Availability: In stock
Link: https://books.toscrape.com/catalogue/scott-pilgrims-precious-little-life-scott-pilgrim-1_987/index.html
------Book------
Title: Rip it Up and Start Again
Price: £35.02
Availability: In stock
Link: https://books.toscrape.com/catalogue/rip-it-up-and-start-again_986/index.html
------Book------
Title: Our Band Could Be Your Life: Scenes from the American Indie Underground, 1981-1991
Price: £57.25
Availability: In stock
Link: https://books.toscrape.com/catalogue/our-band-could-be-your-life-scenes-from-the-american-indie-underground-1981-1991_985/index.html
------Book------
Title: Olio
Price: £23.88
Availability: In stock
Link: https://books.toscrape.com/catalogue/olio_984/index.html
------Book------
Title: Mesaerion: The Best Science Fiction Stories 1800-1849
Price: £37.59
Availability: In stock
Link: https://books.toscrape.com/catalogue/mesaerion-the-best-science-fiction-stories-1800-1849_983/index.html
------Book------
Title: Libertarianism for Beginners
Price: £51.33
Availability: In stock
Link: https://books.toscrape.com/catalogue/libertarianism-for-beginners_982/index.html
------Book------
Title: It's Only the Himalayas
Price: £45.17
Availability: In stock
Link: https://books.toscrape.com/catalogue/its-only-the-himalayas_981/index.html
Total pages scraped: 1
```

## Conclusion

商品一覧、ニュース、研究データのような動的サイトのスクレイピングは難しい場合があります。[Bright Data のツール](https://brightdata.jp/products) は、取り組みのスケールに役立ちます。

- **[Residential Proxies](https://brightdata.jp/proxy-types/residential-proxies):** IP BAN やジオ制限を回避します。
- **[Scraping Browser](https://brightdata.jp/products/scraping-browser):** JavaScript を多用するサイトを簡単に処理できます。
- **[Ready-to-Use Datasets](https://brightdata.jp/products/datasets):** スクレイピングせずに構造化データを取得できます。

これらを jsoup と組み合わせることで、効率的かつ低リスクなデータ抽出が可能です。今すぐ無料でお試しください。