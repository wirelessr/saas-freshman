

```python
class foo(object):
	def get_name(self):
		return sys._getframe(1).f_code.co_name
	def bar(self):
		name = self.get_name()
		print name

f = foo()
f.bar()
```


```python
def logged(func): 
    def with_logging(*args, **kwargs): 
        kwargs['function'] = func.__name__
        return func(*args, **kwargs) 
    return with_logging

class foo(object):
	@logged
	def bar(self, function=''):
		print 'function =', function

f = foo()
f.bar()
```

