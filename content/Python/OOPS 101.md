
### Illustration: classes and inheritance

```python

class Person:

	# class variables
	amount #amount of persons
	
	#constructor
	def __init__(self, name, age, height):
		self.name = name #imagine the object name is self
		self.age  = age
		self.height = height
		Person.amount += 1 # since amount is a class variable, we use Person.amount

	#when we print the object, we can use __str__ to print anything instead of 
	#memory address
	def __str__(self):
		return "Name: {}, Age: {}, Height:{}".format(self.name, self.age,                                                               self.height)
	def growOlder(years):
		self.age += years
	
	#deconstructor
	def __del__(self):
		amount -= 1

# inherited class
class worker(person):
	
	def __init__(self, name, age, height, salary):
		super().__init__(name, age, height, salary) # super() to call parent class
		self.salary = salary
		
	def __str__(self): # overriding the original __str__ fxn in parent Person Class
		return super().__str__() + ", salary:{}".format(self.salary)
		
	def salaryYearly(self):
		return self.salary * 12

mic = Person("mic", 15, 165)
print(Person.amount)
print(mic)
mic.growOlder(3)
print(mic.age)
nic = Person("nic", 18, 187)
print(Person.amount)
print(nic)

bic = worker("bic", 19, 173, 30000)
print(bic.salaryYearly)
print(bic)

del mic
print(Person.amount)
```

### Illustration: operator overloading

(its not present in some langs like java)

```python
class vector:

	def __init__(self, x, y):
		self.x = x
		self.y = y
		
	def __add__(self, vec2):
		return vector((self.x + vec2.x), (self.y + vec2.y))
		
	def __sub__(self, vec2):
		return vector(self.x-vec2.x, self.y-vec2.y)
		
	def __str__(self):
		return "x: {}, y: {}".format(self.x, self.y)
		
v1 = vector(4, 4)
v2 = vector(1,3)
v3 = v1 + v2
print(v3)
```

