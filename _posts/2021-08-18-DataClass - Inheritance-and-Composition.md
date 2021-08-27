---
toc: true
layout: post
description: About Dataclass and objects.
categories: [python, data structures, algorithm]
title: Dataclass - Inhertance and Composition
---

## __Objective__

By the end of this article, you should be able to:

* Construct object in dataclasses.
* Understand and Implment inheritance and composition using dataclasses.
* Understand `field` dataclass.

Before reading this article you must first understand inheritance, composition and some basic python. Visit [realpython's](https://realpython.com/inheritance-composition-python/) for some refresher. Else lets start!.

## __Dataclasses__

In simple terms `Dataclasses` are python classes which is suitable for storing objects data.
Before dataclasses was introducted in python3.7+, one was required to construct an class (object class name Employee) with data `id`, `name` instantiated as such via the `__init__()` constructor method.

```python
class Employee:
    def __init__(self, id: int, name: str):
        self.id = id 
        self.name = name

```

However with dataclasses, one would construct the same class with data above as such

```python
from dataclass import dataclass

@dataclass
class Employee:
    id: int
    name: str
```

As we can see above, Dataclasses automates the creation of `__init__()` constructor method and all we need is to state the object and its type.

We can even go further and create a method in the dataclass method as such

```python
from dataclass import dataclass

@dataclass
class Employee:
    id: int
    name: str

    def employeeinfo(self):
        return f"employee name is {self.name} with id {self.id}"
```

## __Back to Inhertiance__

From [realpython's](https://realpython.com/inheritance-composition-python/) article, "Inheritance is the mechanism you’ll use to create hierarchies of related classes. These related classes will share a common interface that will be defined in the base classes. Derived classes can specialize the interface by providing a particular implementation where applies".

Here, we create an HR systems to process payroll for the company's employees as done in the article.

```python
from typing import List
from dataclasses import dataclass

class PayrollSystem:
    def calculate_payroll(self, employees: List[dataclass]) -> None:
        """Takes a collection of employees and prints their id: str, name: str and check amount: float
        using the .calculate_payroll() method expoed on each employee object.

        Args:
            employees (List): collection of employees
        """
        print("Calculating Payroll")
        print("===================")
        for employee in employees:
            print(f"Payroll for: {employee.id} - {employee.name}")
            print(f"- Check amount: {employee.calculate_payroll()}")
            print("")
```

As noted in the comments above the class `PayrollSystem` has a method `calculate_payroll` which takes collection(i.e. `List`) of employees and displays thier `id`, `name` and checks amount using `calculate_payroll()` method, which is part of the employee object.

We go ahead to implement a base class for employee that handles interface for employee types. This base class will futher be iherited by other classes.

```python
from dataclasses import dataclass
from typing import Union

@dataclass
class Employee:
    """Base class for Employees. Common interface for every employee type.
    Constructed with id: int, name: str
    """
    id: int
    name: str
```

We further create a class `SalaryEmployee` which is a Subclass of the class `Employee`.

```python
from dataclasses import dataclass
from typing import Union

@dataclass
class SalaryEmployee(Employee):
    """Salary Employee Inherits Employee base class and add weekly_salary data.
    """
    weekly_salary: Union[float, int]

    def calculate_payroll(self) -> Union[float, int]:
        """Calulate the payroll and returns pay.
        """
        return self.weekly_salary
```

From the code above, we went past inheriting the base class and add `weekly_salary` data, plus added a method `calculate_payroll` which returns `weekly_salary` data.

Running the code above, we will realize the `dataclass` decorator automatically initialized `weekly_salary` and even initialized the members of the base class.

Guess what, this is one of `dataclass` superpowers!.

We can go further and extend our base class in constucting other classes such as `HourEmployees` and `CommissionEmployee`. We can even add other data to it and create methods as done below.

```python
@dataclass
class HourlyEmployee(Employee):
    """Hourly Employee Inherits Employee base class and add hours_worked and hour_rate data.
    """
    hours_worked: Union[float, int]
    hour_rate: Union[float, int]

    def calculate_payroll(self) -> Union[float, int]:
        """Calulate the payroll and returns pay.
        """
        return self.hours_worked * self.hour_rate


@dataclass
class CommissionEmployee(SalaryEmployee):
    """Comission Employee Inherits SalaryEmployee base class and add commission data.
    """
    commission: int

    def calculate_payroll(self) -> Union[float, int]:
        """Calulate the payroll from SalaryEmployee class using super() and adds commisssion.
        """
        fixed = super().calculate_payroll()
        return fixed + self.commission
```

Testing our implement above we pass them to the `PayrollSystem` class to process payroll;

```python
salary_employee = SalaryEmployee(1, "Sam May", 1500)
hourly_employee = HourlyEmployee(2, 'Jane Doe', 40, 15)
commission_employee = CommissionEmployee(3, 'Kevin Bacon', 1000, 250)

payroll_system = PayrollSystem()
payroll_system.calculate_payroll([salary_employee, hourly_employee, commission_employee])
```

Running the progam and results below;

![Payroll alt <](/images/payroll_system.png)

As you can see, three employee objects was created, passed on the payroll system, which in turn used the `calculate_payroll` method to calculate the payroll for each employee and prints the results.

### __Abstract Base Class and Dataclass in Pyton__

As I noted earlier dataclasses automate automates the creation of `__init__()` constructor method. We can also apply python `abc module` which provide a functionality to prevent creating objects from abstract base classes.

Here is an example below;

```python
from dataclasses import dataclass
from abc import ABC, abstractmethod
from typing import Union

@dataclass
class Employee_(ABC):
    id: int
    name: str

    @abstractmethod
    def calculate_payroll(self):
        pass

@dataclass
class SalaryEmployee(Employee_):
    weekly_salary: Union[float, int]

    def calculate_payroll(self) -> Union[float, int]:
        return self.weekly_salary

@dataclass
class HourlyEmployee(Employee_):
    hours_worked: Union[float, int]
    hour_rate: Union[float, int]

    def calculate_payroll(self) -> Union[float, int]:
        return self.hours_worked * self.hour_rate

@dataclass
class CommissionEmployee(SalaryEmployee):
    commission: int

    def calculate_payroll(self) -> Union[float, int]:
        fixed = super().calculate_payroll()
        return fixed + self.commission
```

Testing our implementation as done above, we get same results below;

![Payroll alt <](/images/payroll_system.png)

We can even go further and extend the derived class to create another derived classes and further add method `work` to it as below.

```python
class Manager(SalaryEmployee):
    def work(self, hours: Union[float, int]) -> str:
        print(f"{self.name} screams and yells for {hours} hours")

class Secretary(SalaryEmployee):
    def work(self, hours: Union[float, int]) -> str:
        print(f"{self.name} expands {hours} hours doing office paperwork")


class SalesPerson(CommissionEmployee):
    def work(self, hours: Union[float, int]) -> str:
        print(f"{self.name} expands {hours} hours on the phone")


class FactoryWorker(HourlyEmployee):
    def work(self, hours: Union[float, int]) -> str:
        print(f"{self.name} manufactures gadgets for {hours} hours")

```

To test our `work` method is working we create productivity platform, which display the work and hours the employee does.

```Python
import Employee
from typing import Union, List

class ProductivitySystem:
    def track(self, employees: List[Employee], hours: Union[float, int]) -> None:
        print("Tracking Employee Productivity")
        print("==============================")
        for employee in employees:
            result = employee.work(hours)
            print(f'{employee.name}: {result}')
        print('')
```

So far, we expect all our implemenation above to work. Testing, we implement a program (as below) which brings all together.

```python
import PayrollSystem
import Manager, Secretary, SalesPerson, FactoryWorker
import ProductivitySystem

manager = Manager(id=1, name='Mary Poppins', weekly_salary=3000)
secretary = Secretary(id=2, name='John Smith', weekly_salary=1500)
sales_guy = SalesPerson(id=3, name='Kevin Bacon', weekly_salary=1000, commission=250)
factory_worker = FactoryWorker(id=4, name='Jane Doe', hours_worked=40, hour_rate=15)


company_employees = [manager, secretary, sales_guy,factory_worker, ]

productivity_system = ProductivitySystem()
productivity_system.track(company_employees, 40)

payroll_system = PayrollSystem()
payroll_system.calculate_payroll(company_employees)

```

Running our code above we get the output below, which indicate all is fine as expected.

![Program output <](/images/program_output.png)

To further test how multiple inheritance work for dataclasses, we will go ahead and create other functions, then perfrom some multiple inheritance here.

We go ahead to add other roles and policies for employees.

```python
from dataclasses import dataclass
from typing import Union

class ManagerRole:
    def work(self, hours: Union[float, int]) -> str:
        return f"screams and yells for {hours} hours."

class SecretaryRole:
    def work(self, hours: Union[float, int]) -> str:
        return f"expands {hours} hours doing office paperwork."

class SalesRole:
    def work(self, hours: Union[float, int]) -> str:
        return f"expands {hours} hours on phone."

class FactoryRole:
    def work(self, hours: Union[float, int]) -> str:
        return f"manufactures gadgets for {hours} hours."

@dataclass
class SalaryPolicy:
    weekly_salary: Union[float, int]

    def calculate_payroll(self) -> Union[float, int]:
        return self.weekly_salary

@dataclass
class HourlyPolicy:
    hours_worked: Union[float, int]
    hour_rate: Union[float, int]

    def calculate_payroll(self) -> Union[float, int]:
        return self.hours_worked * self.hour_rate

@dataclass
class CommissionPolicy(SalaryPolicy):
    commission: int

    def calculate_payroll(self) -> Union[float, int]:
        fixed = super().calculate_payroll()
        return fixed + self.commission
```

We go further to modify some class implemented earlier which make use of multiple inheritance.

```python

from dataclasses import dataclass
from typing import Union

import SalaryPolicy, CommissionPolicy, HourlyPolicy
import ManagerRole, SecretaryRole, SalesRole, FactoryRole


@dataclass
class Employee():
    id: int
    name: str

@dataclass
class Manager(Employee, ManagerRole, SalaryPolicy):
    id: int
    name: str
    weekly_salary: Union[float, int]

    def __post_init__(self):
        SalaryPolicy.__init__(self, self.weekly_salary)
        super().__init__(self.id, self.name)


@dataclass
class Secretary(Employee, SecretaryRole, SalaryPolicy):
    id: int
    name: str
    weekly_salary: Union[float, int]
    
    def __post_init__(self):
        SalaryPolicy.__init__(self, self.weekly_salary)
        super().__init__(self.id, self.name)


@dataclass
class SalesPerson(Employee, SalesRole, CommissionPolicy):
    id: int
    name: str
    weekly_salary: Union[float, int]
    commission: int

    def __post_init__(self):
        CommissionPolicy.__init__(self, self.weekly_salary, self.commission)
        super().__init__(self.id, self.name)


@dataclass
class FactoryWorker(Employee, FactoryRole, HourlyPolicy):
    id: int
    name: str
    hours_worked: Union[float, int]
    hour_rate: Union[float, int]

    def __post_init__(self):
        HourlyPolicy.__init__(self, self.hours_worked, self.hour_rate)
        super().__init__(self.id, self.name)


@dataclass
class TemporarySecretary(Employee, SecretaryRole, HourlyPolicy):
    id: int
    name: str
    hours_worked: Union[float, int]
    hour_rate: Union[float, int]

    def __post_init__(self):
        HourlyPolicy.__init__(self, self.hours_worked, self.hour_rate)
        super().__init__(self.id, self.name)

```

From the code above, I guess you're wondering there is the use of `__post_init__` method and its purpose.

Though `dataclass` decorator automate creation of `__init__` method for us, we would like to get control of this process to tune it for our use. Including `__post_init__` in our class, we can provide other instruction for modifying fields or even instantiate other data.

As you can see in the `Manager` class, we inherited Employee, ManagerRole, SalaryPolicy class and used `__post_init__` to initialize `SalaryPolicy` class. We also used `super()`, which allows us to call methods of the superclass in our subclass.

To test our multple inheritance, we modify our program as done below

```python
import Manager, Secretary, SalesPerson, FactoryWorker, TemporarySecretary
import PayrollSystem
import ProductivitySystem


manager = Manager(id=1, name='Mary Poppins', weekly_salary=3000)
secretary = Secretary(id=2, name='John Smith', weekly_salary=1500)
sales_guy = SalesPerson(id=3, name='Kevin Bacon', weekly_salary=1000, commission=250)
factory_worker = FactoryWorker(id=2, name='Jane Doe', hours_worked=40, hour_rate=15)
temporary_secretary = TemporarySecretary(id=5, name='Robin Williams', hours_worked=40,
                                                   hour_rate=9)

employees = [manager, secretary, sales_guy, factory_worker, temporary_secretary]

productivity_system = ProductivitySystem()
productivity_system.track(employees, 40)

payroll_system = PayrollSystem()
payroll_system.calculate_payroll(employees=employees)
```

Running the code above, as done ealier we get the same output as ealier.

![Program output <](/images/program_output.png)

## __Compostion Here__

From [realpython's](https://realpython.com/inheritance-composition-python/) article, "Composition is an OO design concept that models has a realationship. In composition, a class known as composite contains an object of another class known to as components".

One advantage of composition compared to inheritance is, one component can be changed and such change can rarely affect the composite class. Vice versa a change in the composite class does not affect the component class.

This advantage enables code adaptability and code base changes without introducing problems.

Lets go ahead and implement a Address class which components of an address using `dataclass`;

```python
from dataclasses import dataclass, field
from typing import Optional, Union, Dict

@dataclass
class Address:
    street: str
    city: str
    state: str
    zipcode: str
    street2: Optional[str] = ''
    

    def __str__(self) -> str:
        """Provides pretty response of address."""
        lines = [self.street]
        if self.street2:
            lines.append(self.street2)
        lines.append(f"{self.city}, {self.state} {self.zipcode}")
        return "\n".join(lines)
```

In the code above, we implement the `__str__` method which provides us with pretty implementation of our `Address` object.

Testing our implmentation works, we run the following code

```python
address1 = Address(street="55 main st.", city="concord", state="NH", zipcode="03301")

address2 = Address(street="55 main st.", city="concord", state="NH", zipcode="03301", street2="denso")

print(address1)
print(address2)
```

This gives output

![Address output <](/images/Address_class_output.png)

Since all is working as expected, we modify the `Employee` class by adding the `Address` class as a composite.

```python
import Address
from dataclasses import dataclass
from typing import Union

@dataclass
class Employee():
    """We making the Employee an abstract base class.  There are two side effects here;
    * You telling users of the module that objects of type Employee can't be created.
    * You telling other devs working on the hr module hat if they derive from Employee, the they must 
    override the .calculate_payroll abstract method."""
    id: int
    name: str
    address: Address = None
```

We further go ahead and modify the `PayrollSystem` class to print employees address if present.

```python
from typing import List
from dataclasses import dataclass

class PayrollSystem:
    def calculate_payroll(self, employees: List[dataclass]) -> None:
        """Takes a collection of employees and prints their id: str, name: str and check amount: float
        using the .calculate_payroll() method expoed on each employee object.

        Args:
            employees (List): collection of employees
        """
        print("Calculating Payroll")
        print("===================")
        for employee in employees:
            print(f"Payroll for: {employee.id} - {employee.name}")
            print(f"- Check amount: {employee.calculate_payroll()}")
            if employee.address:
                print("- Sent to:")
                print(employee.address)
            print("")
```

We also modify our program to include `Address` as done below

```python
import Manager, Secretary, SalesPerson, FactoryWorker, TemporarySecretary
import Address
import PayrollSystem
import ProductivitySystem

manager = Manager(id=1, name='Mary Poppins', weekly_salary=3000)
manager.address = Address("121 Admin Rd", "Concord", "NH", "03301")

secretary = Secretary(id=2, name='John Smith', weekly_salary=1500)
secretary.address = Address('67 Paperwork Ave.', 'Manchester',  'NH', '03101')

sales_guy = SalesPerson(id=3, name='Kevin Bacon', weekly_salary=1000, commission=250)

factory_worker = FactoryWorker(id=2, name='Jane Doe', hours_worked=40, hour_rate=15)

temporary_secretary = TemporarySecretary(id=5, name='Robin Williams', hours_worked=40, hour_rate=9)

employees = [manager, secretary, sales_guy, factory_worker, temporary_secretary,]

productivity_system = ProductivitySystem()
productivity_system.track(employees, 40)

payroll_system = PayrollSystem()
payroll_system.calculate_payroll(employees)
```

Running the code above, as done ealier we get the same output as ealier.

![Program output <](/images/program_composition.png)

As we can see the address was printed out if present. As we can see, this design is flexible and we can change `Address` class without having impact on the `Employee` class.

### __Dataclass `field`__

As you can see, using `dataclass` is cool and simple. However, in some cases, we will require or like to customize our `dataclass` field. This is where we the use of `field` comes to play. Also, with the `default` parameter, we can define the default value of our attributes we declared.

Example One: We use `field` to define default values from a function for an attributes:

```python
import uuid
from dataclasses import dataclass, field

def gen_random_id():
    return uuid.uuid1().hex

@dataclass
class Employee:
    name: str
    id: str = field(default_factory=gen_random_id)

# Testing our implementation
Employee(name="kwesi")

#output 
Employee(name='kwesi', id='54073e27060b11ec85e6a44cc81af35c')
```

In the code above, we have a function that generate a random id using `uuid` and `Employee` class attribute `id` uses it.

Example Two: We use `field` to define default values for a class attribute.

```python
import uuid
from dataclasses import dataclass, field

def gen_random_id():
    return uuid.uuid1().hex

@dataclass
class Employee:
    name: str
    id: str = field(default_factory=gen_random_id)
    working_hrs: int = field(default=40)

# Testing our implementation
Employee(name="kwesi")

#output 
Employee(name='kwesi', id='f6b1f661060d11ec9ec9a44cc81af35c', working_hrs=40)
```

In the code above, we extended the function in example one by defining a default value for `working_hrs` attribute.

Example Three: Here things get a bit complicated. We create a new class `EmployeeDB`, which we use `field` to define the class attribute `_employees`. We further use `__post_init__` to modify the  `_employees` data and finally we created a method to display all `_employees` when called.

```python
import uuid
from dataclasses import dataclass, field
from typing import List, Dict

def gen_random_id():
    return uuid.uuid1().hex

@dataclass
class Employee:
    name: str
    id: str = field(default_factory=gen_random_id)
    working_hrs: int = field(default=40)

@dataclass
class EmployeesDB:
    _employees: List[Dict[str, Employee]] = field(default=Dict)

    def __post_init__(self):
        self._employees = [{
            "emp1": Employee("Doe"),
            "emp2": Employee("Jane", working_hrs=50),
            "emp3": Employee("Kwesi", working_hrs=20),
        }]

    @property
    def employees(self):
        return [employee for employee in self._employees]


# Testing our implementation
EmployeesDB().employees

#output 
[{'emp1': Employee(name='Doe', id='b6809f19061111ec86cba44cc81af35c', working_hrs=40),
  'emp2': Employee(name='Jane', id='b680a0cc061111ecb53ca44cc81af35c', working_hrs=50),
  'emp3': Employee(name='Kwesi', id='b680a120061111eca92ca44cc81af35c', working_hrs=20)}]
```

For more about `field` check the official documentation on [dataclass](https://docs.python.org/3/library/dataclasses.html#dataclasses.field).

Check code on [github](https://github.com/emilearthur/OOP_inheritance_composition_dataclass).
