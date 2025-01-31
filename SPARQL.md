# SPARQL Queries for DCAT Catalogs

This document contains a collection of SPARQL queries designed to explore and analyze DCAT (Data Catalog Vocabulary) catalogs. These queries are specifically tested and optimized for two main SPARQL endpoints:

- datos.gob.es SPARQL endpoint: https://datos.gob.es/es/sparql
- European Data Portal SPARQL endpoint: https://data.europa.eu/data/sparql

The queries allow you to extract information about datasets, distributions, data services, and other components of DCAT catalogs.

### SPARQL Queries

#### Catalogs avalaible

```sparql
PREFIX dcat: <http://www.w3.org/ns/dcat#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>

# First, let's check what graphs and catalogs are available
SELECT DISTINCT ?g ?catalog WHERE {
  GRAPH ?g {
    ?catalog a dcat:Catalog .
  }
}
```

#### Catalogs Query
This query selects catalogs, obtaining the title and modification date of each catalog.

```sparql
PREFIX dcat: <http://www.w3.org/ns/dcat#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>

SELECT DISTINCT ?catalog ?title ?modified WHERE {
  ?catalog a dcat:Catalog ;
           dct:title ?title ;
           dct:modified ?modified .
}
ORDER BY DESC(?modified)
```

#### Basic Statistics Query (Optimized for datos.gob.es/EDP)

This query efficiently obtains basic statistics for a specific catalog using subqueries:

Catalog URIs of Spain:
* EDP: `http://data.europa.eu/88u/catalogue/datos-gob-es`
* datos.gob.es: `https://datos.gob.es`

>[!NOTE]
> Replace the catalog URI `<http://data.europa.eu/88u/catalogue/datos-gob-es>` with your desired catalog URI.

```sparql
PREFIX dcat: <http://www.w3.org/ns/dcat#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>

SELECT 
  (?datasets as ?Datasets)
  (?distributions as ?Distributions)
  (?dataServices as ?DataServices)
  (?publishers as ?Publishers)
WHERE {
  { 
    # Count datasets
    SELECT (COUNT(DISTINCT ?dataset) as ?datasets) 
    WHERE {
      <http://data.europa.eu/88u/catalogue/datos-gob-es> dcat:dataset ?dataset .
    }
  }
  { 
    # Count distributions
    SELECT (COUNT(DISTINCT ?distribution) as ?distributions) 
    WHERE {
      <http://data.europa.eu/88u/catalogue/datos-gob-es> dcat:dataset ?dataset .
      ?dataset dcat:distribution ?distribution .
    }
  }
  {
    # Count data services
    SELECT (COUNT(DISTINCT ?dataService) as ?dataServices)
    WHERE {
      {
        # Independent DataServices
        <http://data.europa.eu/88u/catalogue/datos-gob-es> dcat:dataset ?dataService .
        ?dataService a dcat:DataService .
      }
      UNION
      {
        # Linked DataServices
        <http://data.europa.eu/88u/catalogue/datos-gob-es> dcat:dataset ?dataset .
        ?dataset dcat:distribution ?dist .
        ?dist dcat:accessService ?dataService .
      }
    }
  }
  {
    # Count publishers
    SELECT (COUNT(DISTINCT ?publisher) as ?publishers)
    WHERE {
      <http://data.europa.eu/88u/catalogue/datos-gob-es> dcat:dataset ?dataset .
      ?dataset dct:publisher ?publisher .
    }
  }
}
```

#### Datasets Query

This query selects datasets from a specific catalog, obtaining the title and modification date of each dataset.

```sparql
PREFIX dcat: <http://www.w3.org/ns/dcat#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>

SELECT DISTINCT ?dataset ?title ?modified WHERE {
  <https://datos.gob.es> a dcat:Catalog ;
    dcat:dataset ?dataset .
  ?dataset dct:title ?title ;
          dct:modified ?modified .
  FILTER(LANG(?title) = "es")
}
ORDER BY DESC(?modified)
```

#### Distributions Query

This query selects distributions from a specific dataset, obtaining the format and access URL of each distribution.

```sparql
PREFIX dcat: <http://www.w3.org/ns/dcat#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>

SELECT DISTINCT ?distribution ?format ?accessURL WHERE {
  ?dataset a dcat:Dataset ;
           dcat:distribution ?distribution .
  ?distribution dct:format ?format ;
                dcat:accessURL ?accessURL .
}
ORDER BY ?format
```

#### Data Services Query

This query selects data services from a specific catalog, obtaining the title and description of each data service.

```sparql
PREFIX dcat: <http://www.w3.org/ns/dcat#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>

SELECT DISTINCT ?dataService ?title ?description WHERE {
  ?dataService a dcat:DataService ;
               dct:title ?title ;
               dct:description ?description .
  FILTER(LANG(?title) = "es")
}
ORDER BY ?title
```

#### Resources Query

This query selects resources from a specific catalog, obtaining the title and modification date of each resource.

```sparql
PREFIX dcat: <http://www.w3.org/ns/dcat#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>

SELECT DISTINCT ?resource ?title ?modified WHERE {
  ?resource a dcat:Resource ;
            dct:title ?title ;
            dct:modified ?modified .
  FILTER(LANG(?title) = "es")
}
ORDER BY DESC(?modified)
```

### High-Value Datasets (HVD) Queries

#### Find Catalogs with HVD Resources

This query first identifies catalogs containing HVD resources and then lists their datasets:

```sparql
PREFIX dcat: <http://www.w3.org/ns/dcat#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX r5r: <http://data.europa.eu/r5r/>

SELECT DISTINCT ?catalog ?catalogTitle ?dataset ?datasetTitle WHERE {
  # Find catalogs with HVD datasets
  ?catalog a dcat:Catalog ;
          dct:title ?catalogTitle ;
          dcat:dataset ?dataset .
          
  # Get datasets with HVD legislation
  ?dataset r5r:applicableLegislation <http://data.europa.eu/eli/reg_impl/2023/138/oj> ;
          dct:title ?datasetTitle .
}
ORDER BY ?catalogTitle ?datasetTitle
```

#### Find HVD Resources in a Catalog

This query lists HVD datasets, their distributions and services in a catalog:

```sparql
PREFIX dcat: <http://www.w3.org/ns/dcat#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX r5r: <http://data.europa.eu/r5r/>

SELECT DISTINCT ?dataset (SAMPLE(?title) as ?datasetTitle) ?distribution ?service 
WHERE {
  # Find HVD datasets in the catalog
  <http://data.europa.eu/88u/catalogue/datos-gob-es> a dcat:Catalog ;
          dcat:dataset ?dataset .
          
  # Get dataset title and HVD legislation
  ?dataset r5r:applicableLegislation <http://data.europa.eu/eli/reg_impl/2023/138/oj> ;
          dct:title ?title .
          
  # Optionally get related distributions and services
  OPTIONAL {
    ?dataset dcat:distribution ?distribution .
    ?distribution r5r:applicableLegislation <http://data.europa.eu/eli/reg_impl/2023/138/oj> .
    
    OPTIONAL {
      ?distribution dcat:accessService ?service .
      ?service r5r:applicableLegislation <http://data.europa.eu/eli/reg_impl/2023/138/oj> .
    }
  }
}
GROUP BY ?dataset ?distribution ?service
ORDER BY ?datasetTitle
```

#### Count HVD Resources

This query counts the number of HVD datasets, distributions and services:

```sparql
PREFIX dcat: <http://www.w3.org/ns/dcat#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX r5r: <http://data.europa.eu/r5r/>

SELECT 
  (COUNT(DISTINCT ?dataset) as ?hvdDatasets)
  (COUNT(DISTINCT ?distribution) as ?hvdDistributions)
  (COUNT(DISTINCT ?service) as ?hvdServices)
WHERE {
  # HVD Datasets
  <http://data.europa.eu/88u/catalogue/datos-gob-es> dcat:dataset ?dataset .
  ?dataset r5r:applicableLegislation <http://data.europa.eu/eli/reg_impl/2023/138/oj> .
  
  OPTIONAL {
    # HVD Distributions
    ?dataset dcat:distribution ?distribution .
    ?distribution r5r:applicableLegislation <http://data.europa.eu/eli/reg_impl/2023/138/oj> .
  }
  
  OPTIONAL {
    # HVD Services (both direct and through distributions)
    {
      ?distribution dcat:accessService ?service .
      ?service r5r:applicableLegislation <http://data.europa.eu/eli/reg_impl/2023/138/oj> .
    }
    UNION
    {
      ?service dcat:servesDataset ?dataset .
      ?service r5r:applicableLegislation <http://data.europa.eu/eli/reg_impl/2023/138/oj> .
    }
  }
}
```

>[!NOTE]
> These queries are based in: https://dataeuropa.gitlab.io/data-provider-manual/hvd/Reporting_guidelines_for_HVDs/ 