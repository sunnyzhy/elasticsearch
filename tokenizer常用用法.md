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

例如英语单词 quick，5 种长度下的 ngram：

|length|分词结果|
|--|--|
|1|q、u、i、c、k|
|2|qu、ui、ic、ck|
|3|qui、uic、ick|
|4|quic、uick|
|5|quick|

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
			"label": {
				"type": "text",
				"analyzer": "ngram_analyzer",
				"search_analyzer": "ngram_analyzer",
				"index": true
			}
		}
	}
}
```

# Edge NGram tokenizer

这个分词和 nGram 非常的类似。它会固定词语开始的一边滑动窗口，它的结果取决于 n 的长度设置。

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

例如英语单词 quick，Edge NGram之后的结果：

|length|分词结果|
|--|--|
|1|q|
|2|qu|
|3|qui|
|4|quic|
|5|quick|

**因此可以发现，在使用 Edge NGram 建索引时，一个单词会生成好几个索引，而这些索引一定是从首字符开始的。**

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
			"label": {
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
