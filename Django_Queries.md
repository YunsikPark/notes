# Django_Queries

## Making queries

데이터 모델을 만들면 Django는 자동으로 객체를 생성, 검색, 업데이트 및 삭제할 수있는 Database_Abstraction API를 제공합니다.[데이터 모델 참고자료](https://docs.djangoproject.com/en/1.11/ref/models/)

```
from django.db import models

class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()

    def __str__(self):              # __unicode__ on Python 2
        return self.name

class Author(models.Model):
    name = models.CharField(max_length=200)
    email = models.EmailField()

    def __str__(self):              # __unicode__ on Python 2
        return self.name

class Entry(models.Model):
    blog = models.ForeignKey(Blog)
    headline = models.CharField(max_length=255)
    body_text = models.TextField()
    pub_date = models.DateField()
    mod_date = models.DateField()
    authors = models.ManyToManyField(Author)
    n_comments = models.IntegerField()
    n_pingbacks = models.IntegerField()
    rating = models.IntegerField()

    def __str__(self):              # __unicode__ on Python 2
        return self.headline
```

## Creating objects

Django는 Python 객체에서 데이터베이스 테이블 데이터를 나타내기 위해 직관적인 시스템을 사용합니다. **모델 클래스**는 **데이터베이스 테이블**을 의미하고 모델클래스의 **인스턴스**는 **데이터베이스 테이블의 자료**를 의미합니다.

객체를 생성하려면 키워드 인자를 사용하여 모델 클래스에 인스턴스를 생성한 다음 `save()`를 호출하여 객체를 데이터베이스에 저장합니다.

```
>>> from blog.models import Blog
>>> b = Blog(name='Beatles Blog', tagline='All the latest Beatles news.')
>>> b.save()
```

위 코드는 SQL의 INSERT와 같은 역할을 합니다. 하지만 기억해두세요! Django는 ```save()```를 호출 하기 전까지는 절대로 데이터베이스상에 저장을 하지 않습니다. 참고로 ```save()```의 리턴값은 ```None``` 입니다.

**참고!**

```save()```를 건너뛰고 저장을 하는 방법은 ```create()```을 사용하면 됩니다. 더 자세한 설명은 [링크](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#django.db.models.query.QuerySet.create)를 참조하시면 됩니다.

**정리!**

- 모델 클래스 = 데이터베이스 테이블
- 모델 클래스 인스턴스 = 테이블 내 자료

## Object의 변경사항 저장하기
방금 배운거서럼 objects의 변경사항을 저장하기 위해선 ```save()```를 사용하면 됩니다. 아래 주어진 **Blog**의 인스턴스인 **b5**는 이미 저장이 된 자료입니다. 이미 저장이 된 자료를 업데이트 하려면 역시 ```save()``` 를 실행하시면 됩니다. 

```
>>> b5.name = 'New name'
>>> b5.save()
```

 즉, 위의 코드는 SQL에서 UPDATE 구문을 한다는 것을 알 수 있습니다. 참고로, ```b5.name = 'New name```이라고 지정한다고 해도 실제로 ```save()```를 호출하지 않으면 데이터베이스 상에서 업데이트가 되지 않습니다.

## Saving ForeignKey and ManyToManyField fields
ForeignKey 필드를 업데이트하는 방법은 일반 필드를 업데이트 하는 방법과 똑같습니다. 아래의 예시에서는 **Entry**와 **Blog**클래스가 이미 데이터베이스 상에 저장이 되어 있으며 이들의 인스턴스들인 blog와 entry 역시 데이터베이스 상에 저장이 되어 있다는 전제하에 진행을 하겠습니다.

```
>>> from blog.models import Entry
>>> entry = Entry.objects.get(pk=1)
>>> cheese_blog = Blog.objects.get(name="Cheddar Talk")
>>> entry.blog = cheese_blog
>>> entry.save()
```

ManyToManyField를 업데이트하는 것은 약간 다르게 작동합니다. 필드에 `add()` 메소드를 사용하여 관계에 레코드를 추가합니다. 이 예제에서는 작성자 인스턴스 joe를 항목 객체에 추가합니다.

```
>>> from blog.models import Author
>>> joe = Author.objects.create(name="Joe")
>>> entry.authors.add(joe)
```

한 번에 ManyToManyField에 여러 레코드를 추가하려면 다음과 같이 `add()` 호출에 여러 인수를 포함시킵니다.

```
>>> john = Author.objects.create(name="John")
>>> paul = Author.objects.create(name="Paul")
>>> george = Author.objects.create(name="George")
>>> ringo = Author.objects.create(name="Ringo")
>>> entry.authors.add(john, paul, george, ringo)
```

