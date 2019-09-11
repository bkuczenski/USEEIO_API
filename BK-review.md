# USEEIO_API
## Expert Review by Brandon Kuczenski

I conducted a thorough review of the `USEEIO_API` resource published by the US EPA. I found the tool to be reasonably well designed, functional, and adequately documented, with minor exceptions noted below.

### Review Task 1

> Test the actual API on at least one server -  the staging server here - [https://smmtool.app.cloud.gov/api](https://smmtool.app.cloud.gov/api)

I performed tests using the following servers:

 - the staging server on `app.cloud.gov`,
 - the local pre-compiled go server,
 - a local python `flask` server.

I tested the following clients:

 - `curl` command-line http client
 - python `requests` library
 - python `useeio_api.client.Client`

I tested the following routes exhaustively:

 - `get_models()`
 - `get_matrix()`
 - `get_matrix_row()`
 - `get_matrix_column()`
 - `get_sectors()`
 - `get_sector()`
 - `calculate()`

All tests were passed with the following **exceptions**.

 * Using the python client, no queries for DQI matrix content were successful.
  - When queried against the cloud server, all failed with `JSONDecodeError` at `client.py` line 75.
  - When queried against either the local go server or the local python server, all failed with `ValueError` at `client.py` line 47.
  - When using the `requests` library, queries for DQI matrices were successful.
  - When using `curl`, any requests for a DQI matrix failed with HTTP 500 "Failed to load matrix" 

 * `Calculate()` method worked correctly against the local servers and the cloud server but failures due to poorly designed input data are not caught.
  - when using the python client, HTTP 400 resulted in failure with `JSONDecodeError` at `client.py` line 66
  - when using python `requests`, HTTP 400 resulted in failure with apparently empty return data.
  - when using curl, HTTP 400 includes a descriptive error message.
  - example of a successful `curl` command:

    curl -X POST -H "Content-Type: application/json" -d '{"perspective": "direct", "demand": [{"sector": "1111b0/fresh wheat, corn, rice, and other grains/us-ga", "amount": 17.5}]}' https://smmtool.app.cloud.gov/api/GAUSEEIO/calculate

See the ipython notebook `client - cloud and local` for limited documentation of these observations.

Recommend that demand vectors be made more accommodating to poorly crafted input data, in particular by accepting codes or indices in place of full sector names when specifying demand vectors.

### Review Task 2

> Review the API docs on https://smmtool.app.cloud.gov

Documentation is terse but complete and apparently correct.  One error was detected: for the `GET /{model}/matarix/{name}` path, query parameters `row` and `col` are both identified as "required" although neither is required, and if both are supplied, only one is honored.  This is implementation-dependent, but in the python server case, the server processes `col` before `row` and returns early if `col` is present.

Being able to specify both `row` and `col` arguments would be a valuable and easy enhancement.

Errors are poorly handled-- when 

Note that when compiling the API documentation from source, `npm` provides the following warnings:

    npm WARN useeio-apidoc@1.0.0 No description
    npm WARN useeio-apidoc@1.0.0 No repository field.
    npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.2.9 (node_modules/fsevents):
    npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.2.9: wanted {"os":"darwin","arch":"any"} (current: {"os":"linux","arch":"x64"})

Recommend the auto-generated API documentation include a link to the Github Wiki AND/OR the Github `data_format` page.

### Review Task 3

> Review the README and Wiki in the repository https://github.com/USEPA/useeio_api/

`README` file is adequate.

Wiki Build section is hasty, particularly when it comes to the user specified `models.csv` and `demands` inputs.  The `models.csv` file could easily be managed by the `matio.Export()` class.

The `prepare_demands.py` script is not shippable because it requires the user to hard-code paths and because the function / purpose of the script is not specified anywhere.  The script apparently accepts as input a `demands.csv` file and generates as outputs a `demands` folder containing JSON files.

On the wiki `Build` page, the hyperlink to `standard iomb demand format` is mis-targeted to [https://github.com/USEPA/IO-Model-Builder/blob/master/doc/data_format.md#demand-vectors](https://github.com/USEPA/IO-Model-Builder/blob/master/doc/data_format.md#demand-vectors) when in fact it seems like it should point to [https://github.com/USEPA/USEEIO_API/blob/master/doc/data_format.md#demandscsv](https://github.com/USEPA/USEEIO_API/blob/master/doc/data_format.md#demandscsv).

More to the point, it is unclear why "demand files" are needed at all. Demand is the query input, to be supplied by the user. It does not belong as part of the server. What is its purpose? In my opinion, the features and requirements surrounding server-side `demand` specifications should be removed.

Deployment instructions were not tested except to run the pre-compiled app binary and the python `flask` server.

Testing instructions should remove references to `PyCharm` and instead simply reference python's built-in unittesting code.  The documentation included in [https://github.com/USEPA/USEEIO_API/blob/master/python/README.md](https://github.com/USEPA/USEEIO_API/blob/master/python/README.md) is perfectly adequate.

### Review Task 4

> Launch the server on the localhost (either the Python or the Go implementation) with the example model data (attached) and see if you can regenerate the examples

Both the pre-compiled go app and the python flask instance were tested and functional.

### Review Task 5

> Run the python testing suite tests on the localhost and or the staging server

The test suite was successful when run against both the local server and against the cloud server.

### Review Task 6

> Review the apidoc.yaml file in the Swagger Editor to check for compliance with the Swagger 2.0 schema

This was not attempted.

