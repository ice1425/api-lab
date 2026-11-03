# API lab results

## Part 0

- Step 0.1, the number of pizza names: 18
- Step 0.3, status code of the request without a token: 401
- Step 0.4, the name of your pizza: The Bold Quattro Stagioni
- Step 0.12, results of the three runs without a session (optional): Run 1: passed, Run 2: failed (401), Run 3: failed (401)
- Step 0.16, blocks of "how many of the three tools are excluded": 0 tools ([]), 1 tool (["Knife"], ["Scissors"]), 2 tools (["Knife", "Scissors"]), 3 tools (["Knife", "Pizza cutter", "Scissors"]). The original test covered only the 1-tool block. The 3-tool block revealed the defect.

## Defect report

- Title: Excluding all three tools returns 500 with an empty body
- Request: POST https://quickpizza.grafana.com/api/pizza
  Header: Authorization: token <token of a logged-in user>
  Body: {"excludedTools": ["Knife", "Pizza cutter", "Scissors"]}
- Expected result: Status 400 Bad Request with a JSON error message, because no pizza can satisfy the request
- Actual result: Status 500 Internal Server Error with an empty body
- Severity: Medium. A normal customer input crashes the request handler, and the app that calls the API gets no message to show or handle. The rest of the API still works, so it is not High.

## Part A

- Step A4, status codes when the other user reads, changes and deletes:

## Part B

1.
2.
3.
4.
5.
6.
7.
8.
