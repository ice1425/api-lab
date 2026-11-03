# api-lab

API tests for QuickPizza with pytest and requests.

## How to run

pip install -r requirements.txt
pytest -v

## Input domain model for POST /api/ratings

| Characteristic    | Blocks                                      | Values in the tests               | Test that covers each block |
| ----------------- | ------------------------------------------- | --------------------------------- | --------------------------- |
| C1 Value of stars | below 1, 1 to 5 (base), above 5             | 0, 3, 6                           |                             |
| C2 Type of stars  | whole number (base), decimal, text, missing | 3, 3.5, "3", no field             |                             |
| C3 Pizza          | exists (base), does not exist               | a new pizza, 999999999            |                             |
| C4 Token          | the user's own (base), none, wrong length   | from auth, no header, token short |                             |
