# Standard Tokenizer

标准类型的tokenizer对欧洲语言非常友好， 支持Unicode。

|设置|说明|
|--|--|
|max_token_length|最大的token集合,即经过tokenizer过后得到的结果集的最大值。如果token的长度超过了设置的长度，将会继续分，默认255|

# NGram Tokenizer
|设置|说明|默认值|
|--|--|--|
|min_gram|分词后词语的最小长度|1|
|max_gram|分词后数据的最大长度|2|
|token_chars|设置分词的形式，例如数字还是文字。elasticsearch将根据分词的形式对文本进行分词。|[] (Keep all characters)|

token_chars 所接受以下的形式：

|关键字|描述|
|--|--|
|letter|单词，字母 a, b, ï or 京|
|digit|数字3 or 7|
|whitespace|例如 " " or "\n"|
|punctuation|例如 ! or "|
|symbol|例如 $ or √|

使用此分析器时，需要设置两种不同的大小：一种指定要生成的最小 ngrams（min_gram设置），另一种指定要生成的最大 ngrams。

在"spaghetti"示例中，如果您指定 min_gram 为 2 且 max_gram 为 3，则您将获得前两个示例中的组合标记：
```
sp, spa, pa, pag, ag, agh, gh, ghe, he, het, et, ett, tt, tti, ti
```

如果你要将 min_gram 设置为 1 并将 max_gram 设置为 3，那么你将得到更多的标记，从 s，sp，spa，p，pa，pag，a，....开始。

以这种方式分析文本具有一个有趣的优点。 当你查询文本时，你的查询将以相同的方式被分割成文本，所以说你正在寻找拼写错误的单词"spaghety"。搜索这个的一种方法是做一个 fuzzy query，它允许你指定单词的编辑距离以检查匹配。 但是你可以通过使用 ngrams 来获得类似的行为。 让我们将原始单词（spaghetti）生成的 bigrams 与拼写错误的单词（spaghety）进行比较：
```
"spaghetti" 的 bigrams：sp，pa，ag，gh，he，et，tt，ti

"spaghety" 的 bigrams：sp，pa，ag，gh，he，et，ty
```

您可以看到六个 token 重叠，因此当查询包含"spaghety"时，其中带有"spaghetti"的单词仍然匹配。请记住，这意味着您可能不打算使用的原始"spaghetti"单词更多的单词 ，所以请务必测试您的查询相关性！

ngrams 做的另一个有用的事情是允许您在事先不了解语言时或者当您使用与其他欧洲语言不同的方式组合单词的语言时分析文本。 这还有一个优点，即能够使用单个分析器处理多种语言，而不必指定。

示例：
```json
PUT /_template/template_zz
{
	"index_patterns": "zz_*",
	"settings": {
		"number_of_shards": 5,
		"number_of_replicas": 1,
		"analysis": {
			"analyzer": {
				"ngram_analyzer": {
					"type": "custom",
					"tokenizer": "ngram_tokenizer",
					"filter": "lowercase"
				}
			},
			"tokenizer": {
				"ngram_tokenizer": {
					"type": "nGram",
					"min_gram": "2",
					"max_gram": "2",
					"token_chars": ["letter", "digit", "symbol","punctuation"]
				}
			}
		}
	},
	"mappings": {
		"_source": {
			"enabled": true
		},
		"properties": {
			"id": {
				"type": "long",
				"index": true
			},
			"name": {
				"type": "text",
				"analyzer": "ngram_analyzer",
				"search_analyzer": "ngram_analyzer",
				"index": true
			}
		}
	}
}
```

查询：
```json
GET /yy_1/_search
{
  "query": {
    "match_phrase": {
      "name": "xxx#yy"
    }
  }
}
```

# Edge NGram tokenizer

常规 ngram 拆分的变体称为 edge ngrams，仅从前沿构建 ngram。 

|设置|说明|Default value|
|--|--|--|
|min_gram|分词后词语的最小长度|1|
|max_gram|分词后词语的最大长度|2|
|token_chars|设置分词的形式，例如，是数字还是文字。elasticsearch将根据分词的形式对文本进行分词。|[] (Keep all characters)|

token_chars 所接受的以下形式：

|关键字|描述|
|--|--|
|letter|单词，字母 a, b, ï or 京|
|digit|数字3 or 7|
|whitespace|例如 " " or "\n"|
|punctuation|例如 ! or "|
|symbol|例如 $ or √|

在"spaghetti"示例中，如果将 min_gram 设置为 2 并将 max_gram 设置为 6，则会获得以下标记：
```
sp, spa, spag, spagh, spaghe
```

您可以看到每个标记都是从边缘构建的。 这有助于搜索共享相同前缀的单词而无需实际执行前缀查询。 如果你需要从一个单词的后面构建 ngrams，你可以使用 side 属性从后面而不是默认前面获取边缘。

示例：
```json
PUT /_template/template_yy
{
	"index_patterns": "yy_*",
	"settings": {
		"number_of_shards": 5,
		"number_of_replicas": 1,
		"index.max_ngram_diff": 1,
		"analysis": {
			"analyzer": {
				"ngram_analyzer": {
					"type": "custom",
					"tokenizer": "ngram_tokenizer"
				}
			},
			"tokenizer": {
				"ngram_tokenizer": {
					"type": "edge_ngram",
					"min_gram": "2",
					"max_gram": "32",
					"token_chars": ["letter", "digit","symbol","punctuation"]
				}
			}
		}
	},
	"mappings": {
		"_source": {
			"enabled": true
		},
		"properties": {
			"id": {
				"type": "long",
				"index": true
			},
			"name": {
				"type": "text",
				"analyzer": "ngram_analyzer",
				"search_analyzer": "ngram_analyzer",
				"index": true
			}
		}
	}
}
```

# Keyword Tokenizer
keyword  类型的tokenizer 是将一整块的输入数据作为一个单独的分词。

|设置|说明|
|--|--|
|buffer_size|term buffer 的大小. 默认是 to 256.|
|Letter Tokenizer|一个  letter 类型的tokenizer分词是在非字母的环境中将数据分开。也就是说，这个分词的结果可以是一整块的的连续的数据内容 .注意, 这个分词对欧洲的语言非常的友好，但是对亚洲语言十分不友好。|
|Lowercase Tokenizer|一个 lowercase 类型的分词器可以看做Letter Tokenizer分词和Lower case Token Filter的结合体。即先用Letter Tokenizer分词，然后再把分词结果全部换成小写格式。|

# Whitespace Tokenizer

whitespace 类型的分词将文本通过空格进行分词。

# Pattern Tokenizer

一个 pattern类型的分词可以利用正则表达式进行分词。 

|设置|说明|
|pattern|正则表达式的pattern，默认是 \W+.|
|flags|正则表达式的 flags.|
|group|哪个group去抽取数据。 默认是 -1 (split).|

IMPORTANT: 正则表达式应该和 token separators相匹配, 而不是 tokens 它们本身.

使用elasticsearch 不同语言的API 接口时，不必care字符转译问题。

group 设置为-1 (默认情况下) 等价于"split"。

For example, if you have:
```java
pattern = '([^']+)'
group   = 0
input   = aaa 'bbb' 'ccc'
```

the output will be two tokens: 'bbb' and 'ccc' (including the ' marks). With the same input but using group=1, the output would be: bbb and ccc (no ' marks).

# UAX Email URL

uax_url_email 类型的分词器和standard 类型的一十分类似，但是是分的  emails 和url。

|设置|说明|
|--|--|
|max_token_length|经过此分词器后所得的数据的最大长度。 默认是 255.|

# Path Hierarchy Tokenizer

path_hierarchy 路径分词器。

例如有如下数据:
```
/something/something/else
```

经过该分词器后会得到如下数据 tokens:
```
/something
/something/something
/something/something/else
```

|设置|说明|
|--|--|
|delimiter|分隔符，默认 /.|
|replacement|一个选择替代符。 默认是delimiter.|
|buffer_size|缓存buffer的大小, 默认是 1024.|
|reverse|是否将分词后的tokens反转, 默认是 false.|
|skip|Controls initial tokens to skip, defaults to 0.|

# Classic Tokenizer

可以说是为英语而生的分词器. 这个分词器对于英文的首字符缩写、 公司名字、 email 、 大部分网站域名.都能很好的解决。 但是, 对于除了英语之外的其他语言，都不是很好使。

|设置|说明|
|max_token_length|分词后token的最大长度。 默认是 255.|
