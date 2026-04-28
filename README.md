# Silverstripe Forager OpenSearch Provider

OpenSearch indexing provider for [Silverstripe Forager](https://github.com/silverstripeltd/silverstripe-forager).

This module indexes content into an [OpenSearch](https://opensearch.org) cluster using
[opensearch-project/opensearch-php](https://github.com/opensearch-project/opensearch-php).

This module does not provide search query APIs. It is responsible for indexing only.

## Requirements

- PHP 8.3+
- Silverstripe Framework 6.2+
- silverstripe/silverstripe-forager 2.x

## Installation

```bash
composer require somardigital/silverstripe-forager-opensearch
```

## OpenSearch Client Authentication

Authentication mode is selected automatically:

- In `SS_ENVIRONMENT_TYPE=dev`, the client uses basic authentication.
- In non-dev environments, the client uses AWS IAM SigV4 authentication.

If you need to override this behavior, set `auth_type` via Injector configuration.

### Shared Environment Variables

```env
OPENSEARCH_ENDPOINT="https://localhost:9200"   # Required
OPENSEARCH_SSL_VERIFICATION="true"             # Optional, defaults to true
OPENSEARCH_INDEX_VARIANT="dev"                 # Optional
```

`OPENSEARCH_INDEX_VARIANT` is injected into Forager v2 `IndexConfiguration::indexPrefix` by this module.

### Local Basic Auth

```env
OPENSEARCH_ENDPOINT="https://localhost:9200"
OPENSEARCH_USERNAME="admin"
OPENSEARCH_PASSWORD="secret"
OPENSEARCH_SSL_VERIFICATION="false" # Optional when using self-signed certificates
```

### AWS SigV4 Auth

```env
OPENSEARCH_ENDPOINT="https://search-domain.region.es.amazonaws.com"
OPENSEARCH_AWS_REGION="ap-southeast-2"
OPENSEARCH_AWS_SERVICE="es" # Optional, defaults to "es". Use "aoss" for OpenSearch Serverless.
```

For SigV4 authentication, the AWS SDK for PHP must be available. If explicit credentials are not provided,
the AWS default credential provider chain is used.

Security note: Ensure your OpenSearch endpoint uses HTTPS so credentials are encrypted in transit.

## Configuring Index Mappings

OpenSearch mapping types supported by this module:

- `text` (default)
- `alias`
- `binary`
- `boolean`
- `date`
- `float`
- `geo_point`
- `integer`
- `keyword`
- `long`
- `point`
- `object`
- `nested`

Additional supported mapping options:

- `fields`
- `format`
- `ignore_above`
- `ignore_malformed`
- `index`
- `meta`
- `path`
- `properties`
- `store`
- `term_vector`

Example:

```yaml
SilverStripe\Forager\Service\IndexConfiguration:
  indexes:
    myindex:
      includeClasses:
        SilverStripe\CMS\Model\SiteTree:
          fields:
            title: true
            summary_field:
              property: SummaryField
              options:
                type: text
          settings:
            analysis: {}
```

Note: Changing field names or field types often requires a full reindex.
Fields cannot be deleted, so renaming one will leave any previously created fields around.

## Indexing File Content

Use [silverstripe/textextraction](https://docs.silverstripe.org/en/6/optional_features/textextraction/) to extract file content:

```yaml
SilverStripe\Forager\Service\IndexConfiguration:
  indexes:
    myindex:
      includeClasses:
        SilverStripe\CMS\Model\SiteTree:
          fields:
            pdf_example:
              property: PdfExample.FileContent
              options:
                type: text
```

## Forager v2 API Note

If you interact with the indexing service directly in PHP, Forager v2 requires `indexSuffix` as the first
argument for document operations such as `addDocument()`, `removeDocument()`, `addDocuments()`, and `removeDocuments()`.

## Additional Documentation

- [Forager configuration docs](https://github.com/silverstripeltd/silverstripe-forager/blob/main/docs/en/02_configuration.md)
- [Forager customisation docs](https://github.com/silverstripeltd/silverstripe-forager/blob/main/docs/en/05_customising.md)
- [Forager v2 migration notes](https://github.com/silverstripeltd/silverstripe-forager/blob/main/docs/en/12_v2_migration.md)
