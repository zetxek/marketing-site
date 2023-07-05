---
title: Define custom Data Sources
publishedDate: '2023-05-12T21:00:00.0Z'
description: Create custom Data Sources.
---

## Introduction

Roadie Tech Insights lets you define arbitrary Data Sources. Typically, custom Data Sources rely on calling an API or reading a configuration file in a repository. When adding a Data Source, you’ll also specify which Facts are extracted from said source.

To add a Data Source, click on the “Add Data Source” button in the Data Sources page.

![Add Data Source](./data-sources-overview.png)

To set up a Data Source, you will, firstly, need to enter general information such as name and a description. After that you will be need to define a Data Provider and which fields will be extracted as Facts from the data. Let’s start overviewing the options available for setting up a Data Provider:

## Setting up data provider

![Add Data Provider](./data-provider-step.png)

1 - You must specify a type for that new Data Source. Roadie provides few different types of data provider configurations:
  1. The _HTTP_ type lets you connect to an external API to pull in data
  2. _Component repository file_ type lets you extract data from a file path in the corresponding repository of a given Component in your Catalog
  3. _Component repository directory_ type allows you to extract a list of files from the repository

2 - Set additional configuration options depending on the type of the data provider
  1. For _HTTP_ type select a proxy from the provided dropdown and append a path extension to configure the URL the HTTP call should be made. The path extension should be input without the preceding slash. 
  2. For _Component repository file_ configure the path to the file you want to extract data from in repositories, starting from the root. This can be anything from JSON files to YAML files.
  3. For _Component repository_ configure the root folder where you want to list files from. To identify the repository root, you can use `.`. 

3 - Try out what would be the response when testing specific entity from the location you have provided. If you were to get the `package.json` from a `sample-service` component, the Data Source would get something like this:

![Dry run](./dry-run-result.png)

Now that you have data, let’s define what Facts interest you. You’ll do this through the Fields Extraction section.

## Fields extraction

![Fields extraction](./field-extraction.png)

4 - Data retention refers to maximum number of items or duration on how long to keep them before they are automatically removed from the database.

5 - Choose a parser to extract a Fact from the data obtained before. For the type “Component repository file” this can be either JSON or Regex parser type, while for “HTTP” data provider type, only JSON is supported. Retrieved YAML files are handled as JSON. Repository directory configuration returns a single value of type Set and the only configurable options are the name and description of the field.  

JSON type of parser uses  [JSONata query syntax](https://jsonata.org/) to extract data from JSON. Regex type uses [ECMAScript syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions) to extract data from text.

6 - If you’re using the JSON parser, specify a path from the root of the object. For example _“version”,_ or “_scripts.test”_. If you’re using the Regex parser, specify a valid expression with a capture group if extracting values. Please note the Regex does not need slashes at the start or the end.

Let's look how we would do it with an example. 

Using **Regex parser type** from the following result:

![Regex result](./dry-run-yaml-result.png)

Retrieving Node version we could write the following Regex:

![Field extraction Regex](./field-extraction-regex.png)

On the other hand, if you were to have the following result:
 ![JSON result](./json-result-type.png)
 
 and wanted to obtain total pages number, we could use the following syntax
 ![JSONata parser](./fact-parser-jsonata.png)


7 - Select the type of the parsed value.
Here you can select among integer, string, boolean, set and datetime types.

'Check facts' button will run dry run upon data source and newly created data source and let you know what would be the result if data source was being run against entity you have provided as a test entity.

If you wish to add more facts you can do so by clicking ‘Add fact’.

After successfully adding a fact you will be able to select kind and type of services to which data source should apply and publish newly created Data source by clicking ‘Save’ button or save a draft, if you are not ready to publish it yet, using 'Save as draft' button.

## Defining targeted entities

![Data Source Entity Filter](./data-source-entity-filter.png)

You should be able to see the created Data Source in the overview screen. If you decide to create a draft Data Source, you will need to publish it in order for others to see it. This can be achieved using actions menu. 


## Running the data source

![Data Source Publish](./publish-data-source.png)

 Newly created Data Sources have a refresh cycle set to 24hours, but you can modify this value in 'Edit' screen, as well as trigger an update manually from the kebab menu.

![Trigger update](./trigger-update.png)

Note that you can’t trigger manual data updates on built-in Data Sources.

## Recipes

Below are few useful recipes that can be helpful when creating custom data sources.

### JSONata

JSONata syntax is a powerful tool that allows you to do manipulation, arithmetic as well as grouping and other transformations to the returned data. You can see all applicable functions and approaches from the [JSONata documentation site](https://docs.jsonata.org/overview). Sometimes it might be useful to iterate faster by using the [JSONata online editor](https://try.jsonata.org/) to see possible transformations to the returned data.

Below are few commonly used recipes that could be helpful.



<details>
<summary><b> Simple HTTP status response example </b></summary>

API response:

```JSON
[
  {
    "id": 1483,
    "user_id": 2977,
    "title": "Stabilis minima turpe conqueror et.",
    "due_on": "2023-01-08T00:00:00.000+05:30",
    "status": "pending"
  },
  {
    "id": 1480,
    "user_id": 2973,
    "title": "Totam bonus quos avarus corrigo annus pecto unde.",
    "due_on": "2023-01-10T00:00:00.000+05:30",
    "status": "completed"
  },
  {
    "id": 1476,
    "user_id": 2968,
    "title": "Torqueo amoveo molestiae depromo adversus texo.",
    "due_on": "2022-12-26T00:00:00.000+05:30",
    "status": "pending"
  }
]
```

Specify a path from the root of the object. For example `[0].user_id`, or `[2].status`.
For complex queries you have the default JSONata functions available. For example for the above data set the following queries would be possible:

- Amount of items with status 'pending': `$count($[status = 'pending'])`
- Amount of items with status not 'pending': `$count($[state != 'pending'])`
- Title of an item that has a user with id 2973: `$[user_id = 2973].title`
- Title of an item that has an id from the entity annotation `tech-insights/random-id-annotation`: `$[id = '{{ metadata.annotations["tech-insights/random-id-annotation"] }}'].title`

</details>


<details>
<summary><b> Mapping entities and using functions</b></summary>

This one is using Tech Insights scorecard as a data source. It is calling the Roadie API using the `/roadie` proxy with an endpoint path `tech-insights/scorecards/entity-results/:scorecardId`.

API response:

```JSON
{
  "results": [
    {
      "entity": "component:default/my-nice-entity",
      "success": 5,
      "failing": 0
    },
    {
      "entity": "component:default/my-bad-entity",
      "success": 1,
      "failing": 4
    },
    {
      "entity": "component:default/my-failure",
      "success": 0,
      "failing": 5
    }
  ]
}
```

- The number of successful checks for entity: `results.$[entity = $lowercase('{{ kind }}:{{metadata.namespace}}/{{metadata.name}}')].success`
- The number of failing checks for entity: `results.$[entity = $lowercase('{{ kind }}:{{metadata.namespace}}/{{metadata.name}}')].failing`
- The percentage of successful checks: `results.$[entity = $lowercase('{{ kind }}:{{metadata.namespace}}/{{metadata.name}}')].success / (results.$[entity = $lowercase('{{ kind }}:{{metadata.namespace}}/{{metadata.name}}')].success + results.$[entity = $lowercase('{{ kind }}:{{metadata.namespace}}/{{metadata.name}}')].failing) * 100`

</details>


### Proxy usage and the Broker

All Roadie HTTP Tech Insights data source are using Roadie Proxy to connect to third party services. You can configure different proxies [using these instructions](/docs/custom-plugins/proxy/). Additionally if you want to connect to services or endpoints within your own infrastructure, you can also use the Broker connectivity to reach your secure services. To do this, you need to first [set up the broker connection](docs/integrations/broker/). 

Since the endpoints tech insights potentially connects to via the broker are unknown to Roadie beforehand, the user needs to construct their own `accept.json` Broker configuration file to connect to internal endpoints. An example configuration file connecting to a self-hosted metrics server mocked out below.

<details>
<summary><b> A dummy example accept.json configuration </b></summary>


```JSON
{
  "private": [
    {
      "//": "Return metrics from an internal tool",
      "method": "GET",
      "path": "/api/get-my-metrics",
      "origin": "http://metrics-server.internal.our-company.com",
      "auth": {
        "scheme": "bearer",
        "token": "${MY_METRICS_SERVICE_AUTH_TOKEN}"
      }
    }
  ],
  "public": [
    {
      "//": "Get broker connection status",
      "method": "GET",
      "path": "/healthcheck"
    }
  ]
}
```

Where the secret `MY_METRICS_SERVICE_AUTH_TOKEN` is defined by an environment value within the running broker service.

</details>

You can set up the broker connection by using `/broker` proxy configuration and defining an endpoint path like `my-broker-token/api/get-my-metrics`. 

With this kind of set up the Tech Insights data source engine uses a broker connection identified by `my-broker-token` and calls an endpoint `/api/get-my-metrics` via the established broker connection. This configuration matches the mock `accept.json` file seen above, meaning that the Tech Insights data source calls an internal service on internal network hosted under address `http://metrics-server.internal.our-company.com/api/get-my-metrics` and returns response from there. This response can then be mapped to more streamlined and easily usable fact data using the JSONata extractor functionality.  

