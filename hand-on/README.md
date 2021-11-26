# Hand on

## API Spec

1. Greeting version #1

   - Request:
     - Path: /api/v1/greeting?name=<yourname>
     - Method: GET
   - Response:
     - Status Code: 200
     - Content-Type: text/html
     - Body: "Hello, <yourname>!"

2. Greeting version #2

   - Request:
     - Path: /api/v2/greeting
     - Method: POST
     - Content-Type: application/json
     - Body: { "name": "<yourname>" }
   - Response:
     - Status Code: 200
     - Content-Type: application/json
     - Body: { "message": "Hello, <yourname>!" }

## Roles

1. Developer
2. QA
3. Operation

## Timeline

1. Create Hello api/v1: on dev
2. Merge to uat
3. Merge dev to uat
4. Move `inline func` to `func HelloText` in dev branch
5. Found a bug on uat
6. Create Hello api/v2: on dev
7. Fixed bug "append `!` to message" on uat
8. Merge fixed bug to dev
9. Merge to main
10. Move message func to greeting package
11. Fixed bug "add `,` after Hello" on main
12. Merge fixed bug to uat and dev
13. Refactor dev and merge dev to uat
