---
title: ✳ How to create a Batch of data from an in-memory Spark or Pandas dataframe
---

import Prerequisites from '../components/prerequisites.jsx'
import WhereToRunCode from '../components/where_to_run_code.md'
import NextSteps from '../components/next_steps.md'
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

This Core Skill document will help you understand the steps of turning an in-memory dataframe and turning into a Batch, one of the primary data objects for Great Expectations.

<Prerequisites>
- You have configured a Datasource
- Have data that you would like to do as Dataframe.
- If you are using Spark, you will want to have a working spark installation.
</Prerequisites>



# Runtime_DataConnector

In order to create a Batch of data from something in-memory we need to have a `RuntimeDataConnector` as part of your datasource configuration. If you dont have one in your configuration make sure to add it.

A RuntimeDataConnector is a special kind of Data Connector that enables you to use a RuntimeBatchRequest to provide a Batch’s data directly at runtime. The RuntimeBatchRequest can wrap either an in-memory dataframe, filepath, or SQL query, and must include batch identifiers that uniquely identify the data (e.g. a run_id from an AirFlow DAG run). The batch identifiers that must be passed in at runtime are specified in the RuntimeDataConnector’s configuration.

[ DESCRIPTION OF WAHT THE COMPONENTS ARE? ]
[LINK_TO_EXISTING_DOC](https://docs.greatexpectations.io/en/latest/guides/how_to_guides/creating_batches/how_to_configure_a_runtime_data_connector.html?highlight=runtimedataconnector)


[ this looks ugly and like it shouldn't belong here ]
```python
  datasource_config = {
    "name": "example_datasource",
    "class_name": "Datasource",
    "module_name": "great_expectations.datasource",
    "execution_engine": {
        "module_name": "great_expectations.execution_engine",
        "class_name": "PandasExecutionEngine",
    },
    "data_connectors": {
        "default_runtime_data_connector_name": {
            "class_name": "RuntimeDataConnector",
            "module_name": "great_expectations.datasource.data_connector",
            "batch_identifiers": ["default_identifier_name"],
        },
    },
}
```

# Pandas [ this will become a Tab ]

Imagine that this is your Dataframe:

```python
df = pd.DataFrame([[1, 2, 3], [4, 5, 6], [7, 8, 9]], columns=["a", "b", "c"])
```

We are going to be turning it into a Batch. This is different other ways that we interact with Datasources because are aren't actually retrieving data from the datasource. Instead we are passing in data that we have in-memory and allowing the GE machinery to turn it into a Batch object.

# Runtime Batch Request

```python

batch_request = RuntimeBatchRequest(
    datasource_name="example_datasource",
    data_connector_name="default_runtime_data_connector_name",
    data_asset_name="my_data",  # This can be anything that identifies this data_asset for you
    runtime_parameters={"batch_data": df},  # df is your dataframe
    batch_identifiers={"default_identifier_name": "something_something"},
)

```

::: What are the components?
  - datasource_name : this is your configured datasource
  - data_connector_name : this is your configured RuntimeDataConnector
  - data_asset_name : this is something you set yourself.
  - runtime_parameters : this is the key component of a RuntimeBatchRequest... here we are passing in `batch_data` and giving it the df itself. This is the way that you are "inputting" the data into the GE machinery.

  - batch_identifiers :
    - you need this to label your Batch.
    - So if you create a Batch with 2 different data_frames
      - you can have one be "batch_id" : 1
      - and another be "batch_id" : 2
    - but the data_connector has to know about it
      - if the data_connector doesn't have it, then you will get an error.

# creating a validator

  - you have to make a expectation suite
  - you have to then you can use a `get_validator()` method to create a validator
    - which is a Batch + GE machinery

```python    
    context.create_expectation_suite(
        expectation_suite_name="test_suite", overwrite_existing=True
    )
    validator = context.get_validator(
        batch_request=batch_request, expectation_suite_name="test_suite"
    )
    print(validator.head())
```
# what can you do with a Batch?

# what can you do next?
  - Expectations
  -
