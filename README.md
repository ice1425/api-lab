# api-lab

API tests for QuickPizza with pytest and requests.

## How to run

pip install -r requirements.txt
pytest -v

## Input domain model for POST /api/ratings

| Characteristic    | Blocks                                      | Values in the tests               | Test that covers each block                                                                                                                               |
| ----------------- | ------------------------------------------- | --------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| C1 Value of stars | below 1, 1 to 5 (base), above 5             | 0, 3, 6                           | below 1: test_stars_out_of_range[0]; 1 to 5: test_base_choice_is_saved; above 5: test_stars_out_of_range[6]                                               |
| C2 Type of stars  | whole number (base), decimal, text, missing | 3, 3.5, "3", no field             | whole number: test_base_choice_is_saved; decimal: test_stars_decimal_is_refused; text: test_stars_text_is_refused; missing: test_stars_missing_is_refused |
| C3 Pizza          | exists (base), does not exist               | a new pizza, 999999999            | exists: test_base_choice_is_saved; does not exist: test_pizza_not_found_is_refused                                                                        |
| C4 Token          | the user's own (base), none, wrong length   | from auth, no header, token short | own: test_base_choice_is_saved; none: test_no_token_is_refused; wrong length: test_short_token_is_refused                                                 |
