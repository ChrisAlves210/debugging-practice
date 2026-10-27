# Debug Log

**Explain how you used the the techniques covered (Trace Forward, Trace Backward, Divide & Conquer) to uncover the bugs in each exercise. Be specific!**

In your explanations, you may want to answer:

- What is the expected vs. actual output?
- If there is a stack trace, what useful information does it contain?
- Which technique did you use, on which line numbers?
- What assumptions did you have about each line of code, and which ones were proven to be wrong?

_Example: I noticed that the program should show pizza orders once a new order is made, and that it wasn't showing any. So, I used the trace forward technique starting on line 13. I discovered the bug on line 27 was caused by a typo of 'pzza' instead of 'pizza'._

_Then I noticed another bug ..._

## Exercise 1

I expected submitting the order form to save the selected pizza and return to the home page, where the new order would be listed. I traced forward from the form submission into `pizza_order_submit` (app.py lines 65-84) and compared the form field names with the values read by the route. The form sends `order_name` and `pizza_size`, but lines 67-68 ask for `name` and `size`, so those values are `None`; only `crust_type` matches. The form also sends repeated `toppings` checkboxes, so I should read them as a list with `getlist`, not as one value.

The first concrete failure is at line 79: the loop walks every `ToppingType`, rather than the toppings the customer selected, and creates `PizzaTopping(topping=...)`, although the model field is named `topping_type` (line 42). The resulting Flask traceback points to the bad constructor argument and explains why the request stops before showing the success message. Tracing backward from the intended order listing also uncovered that line 81 only adds the pizza to the session; it never commits it, so the order would not persist. Finally, line 84 passes `'/'` to `url_for`, which expects the endpoint name `home` (registered at line 53). I had assumed the form and route used matching names, that every enum topping was the selected list, and that adding a record saved it; comparing the fields, model, and database flow disproved those assumptions.

## Exercise 2

I expected the results page to show the requested city, its temperature in the selected units, description, humidity, and wind speed. I traced forward from the weather form to the `/results` route and found that the form submits `city` and `units`, while lines 39-40 read `users_city` and `requested_units`. Thus both values become `None`. The next assumption to check was that those values are sent using OpenWeather's parameter names: line 45 sends the city as `place`, but the current-weather endpoint expects `q`. With a valid API key and network connection, the request therefore does not reliably return the requested city's weather; an error response lacks `name`, so line 52 can raise a `KeyError` instead of rendering the page.

I then traced backward from the temperature shown in `results.html`: it is populated at line 54 from `main['temperature']`, but OpenWeather's response field is `main['temp']`. Even after correcting the form and city parameter, that mismatch would raise another `KeyError`. The useful information in such a traceback is the exact response lookup that failed; the API error payload would also confirm whether the request itself succeeded. I had assumed the browser's field names matched the route, that the API accepted `place`, and that the JSON temperature key was `temperature`; following each value across the form, request, and response disproved those assumptions.

## Exercise 3

I expected `main.py` to print the input list, then its ascending sorted version, and finally the index of `5` (which is 4 in `[1, 2, 3, 4, 5, 6, 7, 9]`). I used divide and conquer to follow `merge_sort` from its midpoint split at `utils.py` lines 9-17 into the merge loops. Running the program prints the original list, then stops before the sorted-list message. The traceback shows repeated recursive calls at line 16 and ends at line 37 with `IndexError: list index out of range`. That narrows the failure to the right-side remainder loop: it checks `j` but reads `right_side[i]`. The loop should read using `j`; tracing the indices forward showed that `i` can already equal the left-list length while right-side values remain.

Continuing the divide-and-conquer trace exposed two more merge assumptions: line 23 chooses the larger item first, which produces descending order, although the binary search expects ascending order; and neither remainder loop (lines 32-38) advances `k`, so leftover values can overwrite the same output position. After those are corrected, tracing forward into `binary_search` finds that line 48 uses `/`, producing a float that cannot index a list at line 51; it needs integer midpoint division. The `while low < high` condition at line 46 can also stop before checking the final remaining element. I had assumed each half was merged in ascending order, that the remainder loop used the right-hand index, and that midpoint division yielded an array index. The traceback and checking the values and indices at each merge/search step showed otherwise.
