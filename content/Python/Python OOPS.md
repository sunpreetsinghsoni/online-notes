
### Illustration:

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
		return "Name: (), Age: (), Height:()".format(self.name, self.age,                                                               self.height)
	def growOlder(years):
		self.age += years
	
	#deconstructor
	def __del__(self):
		amount -= 1

mic = Person("mic", 15, 165)
print(Person.amount)
print(mic)
mic.growOlder(3)
print(mic.age)
nic = Person("nic", 18, 187)
print(Person.amount)
print(nic)

del mic
print(Person.amount)
	
```

