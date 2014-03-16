---
layout: post
title: "alfred workflow의 한글 처리"
description: ""
tags: alfred 한글 python
---
{% include JB/setup %}

alfred workflow를 만들다보면 한글 argument 처리가 잘 안 되는 현상을 발견하게 됩니다.

Mac에서 unicode를 NFD(Normalization Form Decomposition)으로 처리를 해서 생기는 문제입니다. 

- [관련글 - alfredtweet 한글 자소 풀림](http://jmjeong.com/alfredtweet-한글-자소-풀림-패치-2/)

alfred 인자로 받은 string을 NFC로 변경해주면 비교가 가능합니다. 

	import unicodedata

	q=u”{query}”
	q=unicodedata.normalize(‘NFC’, q)

