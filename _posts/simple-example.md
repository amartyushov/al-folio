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