# Functional Parallelism  

Parallelism inspired by functional programming.

### Tools:
- Fork/Join Framework (RecursiveTask)
- Stream API

## Future
Future Tasks:  Tasks with return values.  
Future Objects:  a "handle" for accessing a task's return value.
- two key operations on future object:
  - Assigment:  
  *The content of the future object is constrained to be single assignment (similar to a final variable in Java), and cannot be modified after the future task has returned.*
  - Blocking Read

Implementation with Fork/Join Framework--[RecursiveTask](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/RecursiveTask.html):
- The compute() method of a future task must have a non-void return type, whereas it has a void return type for regular tasks.
- A method call like left.join() waits for the task referred to by object *left* in both cases, but also provides the task’s return value in the case of future tasks.

## Memoization

## Streams [:triangular_flag_on_post:](https://winterbe.com/posts/2014/07/31/java8-stream-tutorial-examples/) :

```java
List<String> myList =
    Arrays.asList("a1", "a2", "b1", "c2", "c1");

myList
    .stream()
    .filter(s -> s.startsWith("c"))
    .map(String::toUpperCase)
    .sorted()
    .forEach(System.out::println);

//the pipeline can be made to execute in parallel by designating the source to be a parallel stream by simply replacing myList.stream() in the above code by myList.parallelStream() or Stream.of(myList).parallel()
```

1. Intermediate Operation:  
*Intermediate operations return a stream so we can chain multiple intermediate operations without using semicolons*  
filter() map() sorted() are intermediate operation.
2. Terminal Operation:  
*Terminal operations are either void or return a non-stream result*  
forEach() is an terminal operation.

## Data Race
def: Read-Write or Write-Write parallelism.  
Determinism:
1. Function Determinism  
   Same input -> Same output
2. Structural Determinism
   Same input -> Same computation gragh

DRF(data race freedom): Function Determinism + Structural Determinism(using the constructs introduced in this course).  
Benign Non-determinism: different outputs are acceptable.

---
## Practice:
```java
	// Sequentially computes the average age of all actively enrolled students using loops
	public double averageAgeOfEnrolledStudentsImperative(final Student[] studentArray) {
		List<Student> activeStudents = new ArrayList<Student>();
		for (Student s : studentArray) {
			if (s.checkIsCurrent()) {
				activeStudents.add(s);
			}
		}
		double ageSum = 0.0;
		for (Student s : activeStudents) {
			ageSum += s.getAge();
		}
		return ageSum / (double) activeStudents.size();
	}

	//compute the average age of all actively enrolled students using parallel streams.
	public double averageAgeOfEnrolledStudentsParallelStream(final Student[] studentArray) {
		double ageAverage = Arrays.stream(studentArray)
				.parallel()
				.filter(student -> student.checkIsCurrent())
				.collect(Collectors.averagingDouble(s -> s.getAge()));
		return ageAverage;
	}
```

```java
	///Sequentially computes the most common first name out of all students that are no longer active in the class using loops.
    public String mostCommonFirstNameOfInactiveStudentsImperative(final Student[] studentArray) {
        List<Student> inactiveStudents = new ArrayList<Student>();
        for (Student s : studentArray) {
            if (!s.checkIsCurrent()) {
                inactiveStudents.add(s);
            }
        }
        Map<String, Integer> nameCounts = new HashMap<String, Integer>();
        for (Student s : inactiveStudents) {
            if (nameCounts.containsKey(s.getFirstName())) {
                nameCounts.put(s.getFirstName(),
                        new Integer(nameCounts.get(s.getFirstName()) + 1));
            } else {
                nameCounts.put(s.getFirstName(), 1);
            }
        }
        String mostCommon = null;
        int mostCommonCount = -1;
        for (Map.Entry<String, Integer> entry : nameCounts.entrySet()) {
            if (mostCommon == null || entry.getValue() > mostCommonCount) {
                mostCommon = entry.getKey();
                mostCommonCount = entry.getValue();
            }
        }
        return mostCommon;
    }

	// compute the most common first name out of all students that are no longer active in the class using parallel streams
	public String mostCommonFirstNameOfInactiveStudentsParallelStream(final Student[] studentArray) {
		return Arrays.stream(studentArray)
				.parallel()
				.filter(student -> !student.checkIsCurrent())
				.collect(Collectors.groupingBy(Student::getFirstName, Collectors.counting()))
				.entrySet()
				.stream()
				.parallel()
				.max(Map.Entry.comparingByValue())
				.get()
				.getKey();
    }
```

```java
	/**
     * Sequentially computes the number of students who have failed the course
     * who are also older than 20 years old. A failing grade is anything below a
     * 65. A student has only failed the course if they have a failing grade and
     * they are not currently active.
	**/
    public int countNumberOfFailedStudentsOlderThan20Imperative(final Student[] studentArray) {
        int count = 0;
        for (Student s : studentArray) {
            if (!s.checkIsCurrent() && s.getAge() > 20 && s.getGrade() < 65) {
                count++;
            }
        }
        return count;
    }

	public int countNumberOfFailedStudentsOlderThan20ParallelStream(final Student[] studentArray) {
        return (int) Arrays.stream(studentArray)
			.parallel()
			.filter(student -> !student.checkIsCurrent() && student.getAge() > 20 && student.getGrade() < 65)
			.count();
    }
```