---
layout: post
title:  "🐍 Django에서 새로운 kwargs를 갖는 모델 필드 만들기 "
date:   2021-03-24 01:02:00 +0900
categories: bug
tags: [python, django]
---

faker 등을 이용한 테스트까지는 잘 될지 모르겠지만, 장고의 `makemigration` 명령어를 실행할 때 오류가 발생함. 

분명 넣어준 매개변수인데 없다고 뜬다거나, verbose_name을 여러번 입력했다고 한다던가.. 하는 문제가 발생하는데 원인은 필드의 `deconstructor` 에서 직접 추가한 매개변수를 인식하지 못하는 문제가 있기 때문이다.

해당 부분의 코드는 이렇게 구현되어 있다

```python
def deconstruct(self):
    # Short-form way of fetching all the default parameters
    keywords = {}
    possibles = {
        "verbose_name": None,
        "primary_key": False,
        "max_length": None,
        "unique": False,
        "blank": False,
        "null": False,
        "db_index": False,
        "default": NOT_PROVIDED,
        "editable": True,
        "serialize": True,
        "unique_for_date": None,
        "unique_for_month": None,
        "unique_for_year": None,
        "choices": None,
        "help_text": '',
        "db_column": None,
        "db_tablespace": None,
        "auto_created": False,
        "validators": [],
        "error_messages": None,
    }
    # ...
```

원인의 해결방법은 간단한데, name, path, args, kwargs를 반환하는 deconstructor를 구현해 주면 된다. 

```python
def deconstruct(self):
		name, path, args, kwargs = super().deconstruct()
		# Only include kwarg if it's not the default
		kwargs['dataclass'] = self._dataclass
		return name, path, args, kwargs
```