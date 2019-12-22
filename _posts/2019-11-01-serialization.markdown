---
layout: post
title: Java serialization
date: 2019-07-15
description: 
---
# Overview  
JDK1.1  
{: .label .label-green }   
Serialization is a process which **translates** a Java **object** into a sequence of **bytes**.
![Serialization](http://www.plantuml.com/plantuml/png/SoWkIImgISaiIKpaqjQ50cq5JA0Da-EgGH8AJYtAJCp9h4eioSpF0yhQmIHhGCgy4iiIWKmCiy1YIPKbgI1b_abPgSabE9GLbnIb9kQ2JGE1wfHQhCfWDDWBseIzu92QbmBsCm00)

## Why do we need it?
- Normally the maximum life time of the object is from the program start till the program end. 
Serialization may help to _keep object alive_ between the program executions. 
- **Serialized** object (as a byte stream) can be saved to the file and **transferred** by the **network**.
- Serialization enables [Java RMI](https://docs.oracle.com/javase/8/docs/technotes/guides/rmi/faq.html) (Remote Method Invocation) to be performed.    

## How to make a class Serializable
The class needs to implement a marker interface to become a Serializable.
```java
public class User implements Serializable {}
```

## Simple code example
The example demonstrates a serializable class, which is being serialized
and deserialized through saving to the file inside of the test. 
 
```java
public class User implements Serializable {
	private static final long serialVersionUID = 1L;
	
	/**
	 * Static field belongs to the class, not to the instance.
	 * So it is not serialized.
	 */
	static String address = "theEarthPlanet";
	private int age;
	private String name;
	
	/**
	 * Transient fields are ignored during serialization.
	 */
	transient int height;

    // getters and setters of the fields
	}
```
Below is a test which performs serialization and deserialization of the
user instance. Pay attention that _transient_ class field `height` is ignored
during the process of serialization.
```java
public class SimpleSerialization {
	
	@Test
	public void whenSerializedAndDeserialized_objectIsTheSame() throws IOException, ClassNotFoundException {
	    // Arrange
		User user = new User();
		user.setAge(20);
		user.setName("theName");
		user.setHeight(180);
		
		String fileName = "theFile.txt";
		writeObjectToFile(user, fileName);
		
		// Act
		Object object = readObjectFromFile(fileName);
		User deserializedUser = (User) object;
		
		// Assert
		assertThat(deserializedUser.getAge()).isEqualTo(user.getAge());
		assertThat(deserializedUser.getName()).isEqualTo(user.getName());
		
		assertThat(deserializedUser.getHeight()).isNotEqualTo(user.getHeight());
	}
	
	
	private Object readObjectFromFile(String fileName) throws IOException, ClassNotFoundException {
		FileInputStream fileInputStream
				= new FileInputStream(fileName);
		ObjectInputStream objectInputStream
				= new ObjectInputStream(fileInputStream);
		Object object = objectInputStream.readObject();
		objectInputStream.close();
		return object;
	}
	
	
	private void writeObjectToFile(User user, String fileName) throws IOException {
		FileOutputStream fileOutputStream
				= new FileOutputStream(fileName);
		ObjectOutputStream objectOutputStream
				= new ObjectOutputStream(fileOutputStream);
		objectOutputStream.writeObject(user);
		objectOutputStream.flush();
		objectOutputStream.close();
	}
}
```
A source code can be found [here](https://github.com/amartyushov/Experiments/blob/master/java/features/serialization/src/test/java/io/mart/SimpleSerialization.java).

## Classes involved in serialization/deserialization
![classes](http://www.plantuml.com/plantuml/png/SoWkIImgAStDuIf8JCvEJ4zLo4eiIzJBp5UevkBYJCv9B2vMy4_AIaqkySmhA2q9BYbAJSm5od5oQaF55O0YAH2cXYONPorC5v9wD9FyIqlGZIhBpqnHA2_AB4c5cWGrEn_PHB0-X9783hAfqKt9By_JnNGh5oT26wP252DhkHnIyrA0OG40)     
Class `ObjectOutputStream` using a method
```java
public final void writeObject(Object o) throws IOException;
```
can write primitive types or graph of objects to an `OutputStream` as a stream of bytes.  
And streams can then be read using `ObjectInputStream` by the method 
```java
public final Object readObject() throws IOException, ClassNotFoundException;
```   
TODO
{: .label .label-red }  
Create a sequence diagram of serialization/deserialization.

## Caveats: Inheritance and Composition
![Inheritance](http://www.plantuml.com/plantuml/png/ZOz1hi9028Rtd88Bj0TuUOkkz00z0NRGaY2J0AQ9yV3ME8lHbJl-_uFaOueapzjL0HQb23mwMJbGhkojUGS4gydeSbdp3oKms8LKxdHIHWhSMBLTpDL-sXPBzZCjZ7E70Vg4_uzn_B3gvXqEfMusD4duhbyc4rlP7tm2)  
When a class implements an interface `java.io.Serializable` all its subclasses
are becoming serializable as well.   
There is an example of inheritance for serialization.  
<span class="fs-3">[example](https://github.com/amartyushov/Experiments/blob/master/java/features/serialization/src/test/java/io/mart/SerializationInheritance.java){: .btn .btn-purple}</span>  
  
***

![Composition](http://www.plantuml.com/plantuml/png/PKzB2iCW4Drx2jS5Su8k0e7ITNE2YPcqmaHX72cK7huHXYfaz-PztjCn2x2KdOpn13dRHqeoLQJtFPMCYYTWHNDPV6Uw9SOi9aH1ti2ZdP43KFZ0GcXCVzdhTnZQobdJnJDVr_-vWt9hUaVNdjAqY-FmV8dJgbuWt0w-LcckS-ilHenhsUa7)  
In case a class is composed of other classes, then each of these classes
has to implement `java.io.Serializable` otherwise an <font color="red">exception</font> `NotSerializableException`
will be thrown during serialization process.  
On the diagram a class `RootClass` can be serialized.  
But during serialization of class `Subclass` an <font color="red">exception</font> `NotSerializableException` is thrown,
because one of the `Subclass` fields is a nonSerializable class 
(and it is **important** that value for this field is set, otherwise a field value is `null` and no <font color="red">exception</font> is thrown during serialization).  
There is an example of composition for serialization.  
<span class="fs-3">[example](https://github.com/amartyushov/Experiments/blob/master/java/features/serialization/src/test/java/io/mart/SerializationComposition.java){: .btn .btn-purple}</span>  
   
***

![Inheritance correct case](http://www.plantuml.com/plantuml/png/JSv12i9030NGTNEAB1MP7a1S5Bo0q1F4DceWpIAJkb1xTmTcG9VvUNp-a8CyraK19gMSyKBE5lW6x18-ILS-uXWkTeVkqBwxFMoDdn-YSz456oq_ku5OiDzeOXpPXLHUT5K6-B_mzEdxsW6rkYGfjMHWcykoBm00)  
There is an edge case when subclass implements `Serializable`, but parent class not. [Link](https://docs.oracle.com/javase/8/docs/api/java/io/Serializable.html) to javadoc.  
Only the **fields of Serializable objects** are written out and restored.  
In current case it means that for `Child` objects, assuming both `value` and `name` were set to the object
only `name` field values will be serialized/deserialized. And there will be no Runtime exceptions
in this case.      
And fields of non-serializable `Parent` class will be initialized using its no-args constructor (public or protected), so this constructor should be accessible to the `Child` class.
There is an example, which demonstrates the edge case for inheritance.  
<span class="fs-3">[example](https://github.com/amartyushov/Experiments/blob/master/java/features/serialization/src/test/java/io/mart/SerializationInheritanceForSubclassCorrectCase.java){: .btn .btn-purple}</span>  

***

![Inheritance incorrect case](http://www.plantuml.com/plantuml/png/JOzDoeD0343NvXHPVXzbF81qOUa52cwxYMEq0sC6Ch6B_jozGnNgCk-HHz9c8xLbZW0-Kivu8MKnV09M3kyK5swmCmqoGNNOxD-N_yCR91bZvD4QFFMvA9JaIATy5PyacoxUacxfyot4ePHBM6ZUTrVromOnrf4yOySQA8O7zP41zXCUNtMzBi9wLY7ERQHb6SfN7m00)  
There is an edge case for inheritance which will throw an <font color="red">exception</font> in runtime.    
So the subclass `Child` is implementing `Serializable`, but its super class not.   
**Plus** there is not default constructor for super class `Parent`, it means that during **deserialization** there is no possibility to
initialize fields from super class and <font color="red">exception</font> **java.io.InvalidClassException** will be thrown in runtime (`serialization` happens without exceptions). 
There is an example, which demonstrates this edge case of throwing exception  
<span class="fs-3">[example](https://github.com/amartyushov/Experiments/blob/master/java/features/serialization/src/test/java/io/mart/SerializationInheritanceForSubclassIncorrectCase.java){: .btn .btn-purple}</span>  

***

## Serial version UID
It is _strongly recommended_ that all serializable classes explicitly declare the **private** field (
this field is not useful for inheritance):  
```java
private static final long serialVersionUID = 42L;
```   
### Why do we need this field?
<ul>
  <li>the serialization runtime <b>associates</b> each serializable <b>class</b> with a <b>version</b> number, so this field is a version number
      <ul>
        <li>if this field is <b>not</b> deliberately <b>specified</b> in serializable class
            <ul>
              <li>=> serialization runtime <b>calculate</b> default serialVersionUID for this class based on
              its <b>attributes</b>, associated access <b>modifiers</b>
              </li>
              <li>=> when you add/modify any field in class, which is already serialized, 
              => class will not be able to recover, because serialVersionUID generated for new class
              and for old serialized are different => <font color="red">exception</font> <i>java.io.InvalidClassException</i>
              is thrown</li>
            </ul>
        </li>
      </ul>
  </li>
  <li> this field is used during <b>deserialization</b> to verify that saved and loaded objects have the same attributes and thus are 
      compatible on serialization 
  </li>
</ul>

There is an example where fragility of `serialVersionUID` is demonstrated.  
<span class="fs-3">[example](https://github.com/amartyushov/Experiments/blob/master/java/features/serialization/src/test/java/io/mart/SerialVersionUIDTest.java){: .btn .btn-purple}</span>  

## Custom serialization
TODO