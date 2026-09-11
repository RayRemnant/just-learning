same family as C, C++
originally desgined for consumer electronics

compiled vs interpreted

.java source code -> java bytecode -> JVM for that OS -> machine code for that OS

can be run into interpreted mode

.java source code -> java bytecode on the run -> JVM -> machine code

control program flow:
same as JS

code structures
private / public / import / package / class / static / final

identifier

variable types
int, double (floating point), boolean, char, String

String is a Class?
has methods
String s = "hello";
s.length()
s.toUpperCase()

Char is a primitive type. Has no methods.
char c = 'a';
// c.length() // not allowed
Character.toUpperCase(c); // allowed, Character is a wrapper class
Character.toLowerCase(c);

var id 101; // can infer the type (but not used much)

class:
major unit of code

public static void main(String[] args) {
// entry point for java programs
}

// variable assignment by switch statement
double value = switch (unit) {
case INCHES -> inches _ 0.39;
case POUNDS -> pounds _ 0.45;
default -> 0;
};

local variables (inside a method)
instance variables (belong to the object)

variable is null by default

String someText; -> null

class Employee {
String firstName;
String lastName;

    void setValue(double newValue){
        value = newValue < 0 ? 0 : newValue
        // or this.value =
        // "recursive object reference"
    }

    String getValue() {
        return value;
        // this.value
    }

}

Overload methods
class Employee {
void paySalary(int value) {}
void paySalary(int anotherValue) {} // parameter name is irrelevant
double paySalary (int value) {} // return type is irrelevant
void paySalary(int value, int bonus)

}

// Default access modifier
// if in same package -> allowed
// if not -> not allowed
// "package" scoped
class Stuff {
int value;

    void setValue(newValue){

    }

}

// defaults
int i; //0
boolean isSomething; //false
String name; //null (and anything else is null)

Constructor overload

public class Stuff {
private String unit;
private double value;

    public Stuff(String unit, double value){
        this.unit = unit;
        this(unit); // calls the constructor below - matches the interface
        this.value = value;
    }

    public Stuff(String unit){
        this(unit, 0); // can't go both ways - infinite loop
        this.unit = unit;
    }

    public Stuff () {
        this.unit = "lmao";
        this("lmao"); // this is preferable so that we have consistent behaviour in an articulated Constructor
        // tho I'd argue it's readibility sucks.
    }

}

Stuff a = new Stuff("km", 2);
Stuff b = new Stuff("m");
Stuff c = new Stuff():
// all are valid -> they "select" the correct constructor

Code Aggregation Structures
Class
basic unit of code

Package
group or related classes

Module -> version 9
group of related packages

Packages
physically stored as folders
no two classes with the same name in a package

classes

- org
- - financial
- - - insurance
- - - banking
- - - investment

in code:
package org.financial.insurance;
public class Health {
//
}

package + class name should be unique
we use packages de facto

Package naming convention:
reverse domain name

Usage:

option 1:
import org.financial.insurance.Health;
public class Insurance {
Health h = new Health();
}

option 2:
import org.financial.insurance.\*; // import all classes
public class Insurance {
Health h = new Health();
}

option 3:

public class Insurance {
org.financial.insurance.Health h = new org.financial.insurance.Health(); // not recommended
}

// implicitly available
// import java.lang.\*;
// like String, System, Object, Exception

Instance vs Class context

class Stuff {
private static double classValue = 0; // belongs to the class
private double instanceValue; // belongs to the instance

    public static double calculate(double value) {
        classValue = value + classValue;
        // using instanceValue is forbidden, can't tell which one
        return classValue;
    }


    public void setInstanceValue(double value) {
        instanceValue = value;
    }

    public double getInstanceValue() {
        return instanceValue;
    }

}

double value = Stuff.calculate(1); // uses the static method of the class, doesn't need an instance

Stuff stuff1 = new Stuff();
stuff1.setInstanceValue(1);
stuff1.calculate(2); //valid, but weird, can't use instance values, only the parameters passed



Inheritance

public class Experiment {
    public String getSummary() {
        return "Experiment";
    }
}

public class BetterExperiment extends Experiment {

    // override annotation is optional, but good practice for readability and avoid typos
    @Override // indicates that we are overriding the getSummary method
    public String getSummary() {

        // calls the parent method
        return super.getSummary() + " but BETTER";
    }
}


The Object class

public class Experiment {
}

// implicitly extends object class

public class Experiment extends Object {
}

these methods are always available, because are from the Object class
- equals()
- hashCode()
- toString()
- getClass()

what happens inside the toString method:

public class Object {
    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
}

// HUH?