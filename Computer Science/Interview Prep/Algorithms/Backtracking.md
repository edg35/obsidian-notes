
### Template

```java
public static boolean isValidState(state) {
	return true;
}

public static void getCandidates(state) {
	return new int[];
}

public static int[] search(state, solutions) {
	if (isValidState) solutions.add(state.copy);
	
	for (int c: getCandidates) {
		state.add(candidate);
		search(state, solutions);
		state.remove(candidate);
	}
}

public static ArrayList<Integer> solve() {
	ArrayList<Integer> solutions = new ArrayList<>();
	HashSet<Integer> state = new HashSet<>();

	search(state, solutions);
	return solutions;
}
```

