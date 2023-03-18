> [中文](https://github.com/SundayWindy/ExcelAlchemy/blob/master/README_cn.md) | English
> 

# ExcelAlchemy User Guide
# 📊 ExcelAlchemy

ExcelAlchemy is a Python library that allows you to download Excel files from Minio, parse user inputs, and generate corresponding Pydantic classes. It also allows you to generate Excel files based on Pydantic classes for easy user downloads.

## Installation

Use pip to install:

```
pip install ExcelAlchemy
```

## Usage

### Generate Excel template from Pydantic class

```python
from excelalchemy import ExcelAlchemy, FieldMeta, ImporterConfig, Number, String
from pydantic import BaseModel

class Importer(BaseModel):
    age: Number = FieldMeta(label='Age', order=1)
    name: String = FieldMeta(label='Name', order=2)
    phone: String | None = FieldMeta(label='Phone', order=3)
    address: String | None = FieldMeta(label='Address', order=4)

alchemy = ExcelAlchemy(ImporterConfig(Importer))
base64content = alchemy.download_template()
print(base64content)

```
* The above is a simple example of generating an Excel template from a Pydantic class. The Excel template will have a sheet named "Sheet1" with four columns: "Age", "Name", "Phone", and "Address". "Age" and "Name" are required fields, while "Phone" and "Address" are optional.
* The method returns a base64-encoded string that represents the Excel file. You can directly use the window.open method to open the Excel file in the front-end, or download it by typing the base64 content in the browser's address bar.
* When downloading a template, you can also specify some default values, for example:
    
```python
from excelalchemy import ExcelAlchemy, FieldMeta, ImporterConfig, Number, String
from pydantic import BaseModel

class Importer(BaseModel):
    age: Number = FieldMeta(label='Age', order=1)
    name: String = FieldMeta(label='Name', order=2)
    phone: String | None = FieldMeta(label='Phone', order=3)
    address: String | None = FieldMeta(label='Address', order=4)

alchemy = ExcelAlchemy(ImporterConfig(Importer))

sample = [
    {'age': 18, 'name': 'Bob', 'phone': '12345678901', 'address': 'New York'},
    {'age': 19, 'name': 'Alice', 'address': 'Shanghai'},
    {'age': 20, 'name': 'John', 'phone': '12345678901'},
]
base64content = alchemy.download_template(sample)
print(base64content)
```
In the above example, we specify a sample, which is a list of dictionaries. Each dictionary represents a row in the Excel sheet, and the keys represent column names. The method returns an Excel template with default values filled in. If a field doesn't have a default value, it will be empty. For example:
* ![image](https://github.com/SundayWindy/ExcelAlchemy/raw/master/images/001_sample_template.png)

### Parse a Pydantic class from an Excel file and create data

```python
import asyncio
from typing import Any

from excelalchemy import ExcelAlchemy, FieldMeta, ImporterConfig, Number, String
from minio import Minio
from pydantic import BaseModel


class Importer(BaseModel):
    age: Number = FieldMeta(label='Age', order=1)
    name: String = FieldMeta(label='Name', order=2)
    phone: String | None = FieldMeta(label='Phone', order=3)
    address: String | None = FieldMeta(label='Address', order=4)


def data_converter(data: dict[str, Any]) -> dict[str, Any]:
    """Custom data converter, here you can modify the result of Importer.dict()"""
    data['age'] = data['age'] + 1
    data['name'] = {"phone": data['phone']}
    return data


async def create_func(data: dict[str, Any], context: None) -> Any:
    """Your defined creation function"""
    # do something to create data
    return True


async def main():
    alchemy = ExcelAlchemy(
        ImporterConfig(
            create_importer_model=Importer,
            creator=create_func,
            data_converter=data_converter,
            minio=Minio(endpoint=''),  # reachable minio address
            bucket_name='excel',
            url_expires=3600,
        )
    )
    result = await alchemy.import_data(input_excel_name='test.xlsx', output_excel_name="test.xlsx")
    print(result)


asyncio.run(main())
```

* The importing function is based on `Minio`, so you need to install Minio and create a bucket to use this functionality for storing the Excel files.
  
* The imported Excel file must be generated by the `download_template()` method, otherwise, it will produce a parsing error.
* In the above example, we define a `data_converter` function, which is used to modify the result of `Importer.dict().` The final result of `data_converter` function will be the parameter of the create_func function. This function is optional if you don't need to modify the data.
* The `create_func` function is used to create data, and the parameter is the result of the data_converter function, and context is None. You can create data, for example, by storing the data in a database.
* The `input_excel_name` parameter of the `import_data()` method is the name of the Excel file in Minio, and the `output_excel_name` parameter is the name of the Excel file with the parsing result in Minio. This file contains all the input data, and if any data fails the parsing, the first column of that data has an error message, and the error-producing cell is highlighted in red.
* The method returns an `ImportResult` type result. You can see the definition of this class in the code. This class contains all the information about the parsing result, such as the number of successfully imported data, the number of failed data, the failed data, etc.
* An example of the importing result is shown in the following image:
![image](https://github.com/SundayWindy/ExcelAlchemy/raw/master/images/002_import_result.png)


### Contributing
If you have any questions or suggestions regarding the ExcelAlchemy library, please raise an issue in [GitHub Issues](https://github.com/SundayWindy/ExcelAlchemy/issues). We also welcome you to submit a pull request to contribute your code.

### License
ExcelAlchemy is licensed under the MIT license. For more information, please see the [LICENSE](https://github.com/SundayWindy/ExcelAlchemy/blob/master/LICENSE) file.