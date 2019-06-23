# Daily Questions

[TOC]

## 1. Map & HashMap 

### 1.1 `map.putIfAbsent()` vs `map.getOrDefault()`

Q: Which one is better? Mainly memory friendly.

```java
// Update a hash map 
Map<Integer, List<Integer>> map = new HashMap<>();

// getOrDefault()
List<Integer> current = map.getOrDefault(1, new ArrayList<>());
current.add(5);
map.put(1, current);

// putIfAbsent()
map.putIfAbsent(1, new ArrayList<>());
map.get(1).put()
```

