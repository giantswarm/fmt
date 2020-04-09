# Conditionals and the Happy Path of a function

In general, the true business logic that a function is concerned with should be on the leftmost indent of that function.

By adjusting your conditionals you are able to achieve this. All indented code within the conditionals then become guard clauses that handle all the exceptional cases that this function was not made to handle.


Dont:
```
function alertTheBoss(isWorkingHours) {
  if (isWorkingHours) {
    alert("Boss, you have an alert!");
  }
}
```

Do:
```
function alertTheBoss(isWorkingHours) {
  if (!isWorkingHours) return;

  alert("Boss, you have an alert!");
}
```
