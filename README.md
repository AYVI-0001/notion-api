<!-- markdownlint-disable MD033 MD045-->

# notion-api

<p align="center">
    <a href="https://pypi.org/project/notion-api"><img alt="PYPI" src="https://img.shields.io/pypi/v/notion-api"></a>
    &nbsp;
    <a href="https://pypi.org/project/notion-api"><img alt="PYPI" src="https://img.shields.io/pypi/status/notion-api"></a>
    &nbsp;
    <img alt="pyversions" src="https://img.shields.io/pypi/pyversions/notion-api"></a>
    &nbsp;
    <img alt="last commit" src="https://img.shields.io/github/last-commit/ayvi-0001/notion-api?color=%239146ff"></a>
    &nbsp;
    <a href="https://developers.notion.com/reference/versioning"><img alt="notion versioning" src="https://img.shields.io/static/v1?label=notion-API-version&message=2022-06-28&color=%232e1a00"></a>
    &nbsp;
</p>
<p align="center">
    <a href="https://github.com/ayvi-0001/notion-api/blob/main/LICENSE"><img alt="license: MIT" src="https://img.shields.io/github/license/ayvi-0001/notion-api?color=informational"></a>
    &nbsp;
    <a href="https://github.com/psf/black"><img alt="code style: black" src="https://img.shields.io/badge/code%20style-black-000000.svg"></a>
    &nbsp;
    <a href="https://pycqa.github.io/isort/"><img alt="code style: black" src="https://img.shields.io/badge/%20imports-isort-%231674b1?style=flat&labelColor=ef8336"></a>

</p>

__Disclaimer: This is an _unofficial_ package and has no affiliation with Notion.so__  

A wrapper for Notion's API, aiming to simplify the dynamic nature of interacting with Notion.  
README contains examples of the main functionality, including: creating Pages/Databases/Blocks, adding/removing/editing properties, retrieving property values, and database queries.  
Some more in-depth walkthroughs can be be found in [`examples/`](https://github.com/ayvi-0001/notion-api/tree/main/examples).  

This library is not complete - new features will continue to be added, and current features may change.

<br>

<div border="0" align="center">
    <table>
        <tr>
            <td align="center"><b>Links: Notion API Updates</b></td>
        </tr>
            <td> <a href="https://developers.notion.com/reference/intro">API Reference</a></td><tr>
        </td>
            <td><a href="https://developers.notion.com/page/changelog">Notion API Changelog </img></a></tr>
            <td> <a href="https://www.notion.so/releases">Notion.so Releases</a></td></tr>
        </tr>
    </table>
</div>

<br>

If you haven't already, you'll need to setup a [Notion Integration](https://www.notion.so/my-integrations) and share specific pages/databases with the integration to interact with the API.  
Information on integration types and setup can be found [here](https://developers.notion.com/docs/getting-started).

---

## Install

```sh
pip install -U notion-api
```

## Usage

```py
import notion

# client will check for 'NOTION_TOKEN' in environment variables.

homepage = notion.Page('773b08ff38b44521b44b115827e850f2')
parent_db = notion.Database(homepage.parent_id)

# client will also look for env var `TZ` to set the default timezone.
# If not found, attempts to find the default timezone, or "UTC".
```

Indexing a page will search for page property values, and indexing a Database will search for property objects.  
A full list can be retrieved for both using;  

- `retrieve()` method for a Page, with the optional `filter_properties` parameter.
- `retrieve` attribute for a Database.

```py
homepage['dependencies']
# {
#     "id": "WYYq",
#     "type": "relation",
#     "relation": [
#         {
#             "id": "7bcbc8e6-e237-434b-bd0d-6b56e044200b"
#         }
#     ],
#     "has_more": false
# }

parent_db['dependencies']
# {
#     "id": "WYYq",
#     "name": "dependencies",
#     "type": "relation",
#     "relation": {
#         "database_id": "f5984a7e-2257-4ab0-9d0a-23ea12324031",
#         "type": "dual_property",
#         "dual_property": {
#             "synced_property_name": "blocked",
#             "synced_property_id": "wx%7DQ"
#         }
#     }
# }
```

**_See usage of retrieving values from a page in [examples/retrieving-property-items.md](https://github.com/ayvi-0001/notion-api/blob/main/examples/retrieving-property-items.md)_**  

Below is a brief example to retrieve the `dependencies` property above from `homepage`.

```py
from notion import propertyitems

related_id: list[str] = propertyitems.relation(homepage.dependencies)
```

```py
>>> related_id
["7bcbc8e6-e237-434b-bd0d-6b56e044200b"]
```

Both Page's and Database's have setters for title/icon/cover.

```py
homepage.title = "new page"
homepage.cover = "https://www.notion.so/images/page-cover/webb1.jpg"
homepage.icon = "https://www.notion.so/icons/alien-pixel_purple.svg"
```

<p align="center"> <img src="https://github.com/ayvi-0001/notion-api/blob/main/examples/images/new_page.png?raw=true"> </p>

---

## Creating Pages/Databases/Blocks

Create objects by passing an existing Page/Database instance to the `create` classmethods.  
The current version of the Notion api does not allow pages/databases to be created to the parent `workspace`.  

```py
new_database = notion.Database.create(
    parent_instance=testpage,
    database_title="Example Database",
    name_column="page", # This is the column containing page names. Defaults to "Name".
    is_inline=True, # can also toggle inline with setters.
    description="Database description can go here.",
)

new_page = notion.Page.create(new_database, page_title="A new database row")
```

Blocks are also created using classmethods. They require a parent instance of either `Page` or `Block` to append the new block too.
The newly created block is returned as an instance of `Block`, which can be used as the parent instance to another nested block.

By default, blocks are appended to the bottom of the parent block.  
To append the block somewhere else other than the bottom of the parent block, use the `after` parameter and set its value to the ID of the block that the new block should be appended after. The block_id used in the `after` paramater must still be a child to the parent instance.  

```py
from notion import properties as prop

original_synced_block = notion.Block.original_synced_block(homepage)

# Adding content to the synced block
notion.Block.paragraph(original_synced_block, [prop.RichText("This is a synced block.")])

# Referencing the synced block in a new page.
notion.Block.duplicate_synced_block(new_page, original_synced_block.id)
```

---

There are few extensions to the `Block` class that have specific functions unique to their block-type.  
Below is an example using `CodeBlock`. The others are `TableBlock`, `EquationBlock`, `RichTextBlock`, and `ToDoBlock`. You can see usage for them in [`examples/block_extensions.md`](https://github.com/ayvi-0001/notion-api/blob/main/examples/block_extensions.md).

```py
code_block = notion.CodeBlock("84c5721d8a954667902a757f0033f9e0")

git_graph = r"""
%%{init: { 'logLevel': 'debug', 'theme': 'default' , 'themeVariables': { 'darkMode':'true', 'git0': '#ff0000', 'git1': '#00ff00', 'git2': '#0000ff', 'git3': '#ff00ff', 'git4': '#00ffff', 'git5': '#ffff00', 'git6': '#ff00ff', 'git7': '#00ffff' } } }%%
gitGraph
       commit
       branch develop
       commit tag:"v1.0.0"
       commit
       checkout main
       commit type: HIGHLIGHT
       commit
       merge develop
       commit
       branch featureA
       commit
"""

code_block.language = prop.CodeBlockLang.mermaid
code_block.code = git_graph
code_block.caption = "Example from https://mermaid.js.org/syntax/gitgraph.html"
```

<p align="center">
    <img src="https://github.com/ayvi-0001/notion-api/blob/main/examples/images/code_commit_diagram.png?raw=true">
</p>

---

**_Example Function: Using `notion.Workspace()` to retrieve a user, and appending blocks in a page to mention user/date._**

```py
def inline_mention(page: notion.Page, message: str, user_name: str) -> None:
    mentionblock = notion.Block.paragraph(
        page,
        [
            prop.Mention.user(
                notion.Workspace().retrieve_user(user_name=user_name),
                annotations=prop.Annotations(
                    code=True, bold=True, color=prop.BlockColor.purple
                ),
            ),
            prop.RichText(" - "),
            prop.Mention.date(
                datetime.now().astimezone(page.tz).isoformat(),
                annotations=prop.Annotations(
                    code=True,
                    bold=True,
                    italic=True,
                    underline=True,
                    color=prop.BlockColor.gray,
                ),
            ),
            prop.RichText(":"),
        ]
    )
    # First method returned the newly created block that we append to here:
    notion.Block.paragraph(mentionblock, [prop.RichText(message)])
    notion.Block.divider(page)
```

```py
homepage = notion.Page("0b9eccfa890e4c3390175ee10c664a35")
inline_mention(page=homepage, message="example", user_name="AYVI")
```

<p align="center">
    <img src="https://github.com/ayvi-0001/notion-api/blob/main/examples/images/example_function_reminder.png?raw=true">
</p>

---

## Add, Set, & Delete: Page property values | Database property objects

The first argument for all database property methods is the name of the property,  
If a property of that name does not exist, then a new property will be created.
If a property of that name already exists, but it's a different type than the method used - then the API will overwrite this and change the property object to the new type.  
The original parameters will be saved if you decide to switch back (i.e. if you change a formula column to a select column, upon changing it back to a formula column, the original formula expression will still be there).

```py
new_database.formula_column("page_id", expression="id()")

new_database.delete_property("url")

new_database.multiselect_column(
    "New Options Column",
    options=[
        prop.Option("Option A", prop.PropertyColor.red),
        prop.Option("Option B", prop.PropertyColor.green),
        prop.Option("Option C", prop.PropertyColor.blue),
    ],
)

# if an option does not already exist, a new one will be created with a random color.
# this is not true for `status` column types, which can only be edited via UI.

new_page.set_multiselect("options", ["option-a", "option-b"])
```

---

## Database Queries

A single `notion.query.PropertyFilter` is equivalent to filtering one property type in Notion.
To build nested filters, use `notion.query.CompoundFilter` and group property filters chained together by `_and(...)` / `_or(...)`.

The database method `query()` will return the raw response from the API.  
The method `query_pages()` will extract the page ID for each object in the array of results, and return a list of `notion.Page` objects.

> note: in v0.6.0, new query methods were added: `query_all()` & `query_all_pages()` - these handle pagination, and instead of providing the `next_cursor` and `has_more` keys,
> they will iterate through all the results, or up to the `max_page_size` paramater. These 2 methods will replace the original ones in a later update.

```py
import os
from datetime import datetime, timedelta
from pytz import timezone

from notion import query

TZ = timezone(os.getenv("TZ", "UTC"))

TODAY = datetime.combine(datetime.today(), datetime.min.time()).astimezone(TZ)
TOMORROW = TODAY + timedelta(1)

query_filter = query.CompoundFilter()._and(
    query.PropertyFilter.date("date", "created_time", "on_or_after", TODAY.isoformat()),
    query.PropertyFilter.date("date", "created_time", "before", TOMORROW.isoformat()),
    query.CompoundFilter()._or(
        query.PropertyFilter.text("name", "title", "contains", "your page title"),
        query.PropertyFilter.text("name", "title", "contains", "your other page title")
    ),
)

query_sort = query.SortFilter(
    [
        query.PropertyValueSort.ascending("your property name"),
        query.EntryTimestampSort.created_time_descending(),
    ]
)

query_result = new_database.query(
    filter=query_filter,
    sort=query_sort,
    page_size=5,
    filter_property_values=["name", "options"],
)
```

---

## Exceptions & Validating Responses

Errors in Notion requests return an object with the keys: 'object', 'status', 'code', and 'message'.
Exceptions are raised by matching the error code and returning the message. For example:

```py
homepage._patch_properties(payload={'an_incorrect_key':'value'})
# Example error object for line above..
# {
#   'object': 'error', 
#   'status': 400, 
#   'code': 'validation_error', 
#   'message': 'body failed validation: body.an_incorrect_key should be not present, instead was `"value"`.'
# }
```

```sh
Traceback (most recent call last):
File "c:\path\to\file\_.py", line 6, in <module>
    homepage._patch_properties(payload={'an_incorrect_key':'value'})
File "c:\...\notion\exceptions\validate.py", line 48, in validate_response
    raise NotionValidationError(message)
notion.exceptions.errors.NotionValidationError: body failed validation: body.an_incorrect_key should be not present, instead was `"value"`.
Error 400: The request body does not match the schema for the expected parameters.
```

Possible errors are:

- `NotionConflictError`
- `NotionDatabaseConnectionUnavailable`
- `NotionGatewayTimeout`
- `NotionInvalidGrant`
- `NotionInternalServerError`
- `NotionInvalidJson`
- `NotionInvalidRequest`
- `NotionInvalidRequestUrl`
- `NotionMissingVersion`
- `NotionObjectNotFound`
- `NotionRateLimited`
- `NotionRestrictedResource`
- `NotionServiceUnavailable`
- `NotionUnauthorized`
- `NotionValidationError`

A common error to look out for is `NotionObjectNotFound`. This error is often raised because your bot has not been added as a connection to the page.

<p align="center">
    <img src="https://github.com/ayvi-0001/notion-api/blob/main/examples/images/directory_add_connections.png?raw=true">  
</p>

By default, a bot will have access to the children of any Parent object it has access too. Be sure to double check this connection when moving pages.  
If you're working on a page that your token has access to via its parent page/database, but you never explicitly granted access to the child page -  and you later move that child page out, then it will lose access.

---
