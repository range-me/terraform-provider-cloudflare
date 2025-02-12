# Provider Design

## API, SDK and Terraform provider Boundaries

The Cloudflare provider implements support for the Cloudflare resources using the [cloudflare-go] library, which in turn, wraps the public API. The API documentation can be found at [api.cloudflare.com] with some supporting developer documentation at [developers.cloudflare.com].com

Where possible, we aim to remove any functionality that isn't explicitly related to the creation, management or deletion of a resource from Terraform instead pushing those responsibilities to the SDK or Cloudflare service itself. Here are some examples of things that **can** live in the Terraform provider:

- Manipulating the API response with a list of strings into either a `TypeList` or `TypeSet` depending on the ordering needs.
- Checking the HTTP response payload or status to determine if a resource should be removed from the state.

And here are some things that **should not** be included in the Terraform provider with examples of preferred approaches:

- Manually handling filtering of resources or data sources. Instead, try to use server side filtering where possible that is configured using `cloudflare-go` and only configure passing the desired attributes from Terraform. If server side filtering isn't possible, encapsulate the logic in `cloudflare-go` and raise a Cloudflare support ticket.
- Complex validation logic that is tied to entitlements, a subscription or a specific product. Instead, consider implementing simple validation checks for faster developer feedback and leave the complex checks to the service and handle the error response accordingly.
- Creating and parsing JSON from resources. Instead, `cloudflare-go` should be managing all structs, interfaces and types.
- Additional formatting of payloads and remapping of fields. Aim to maintain a 1:1 mapping to the public API for your schema which limits duplicating remapping logic in other tools such as [cf-terraforming].


## Data Sources

A separate class of Terraform resource types are [data sources](https://www.terraform.io/docs/language/data-sources/). These are typically intended as a configuration method to lookup or fetch data in a read-only manner. Data sources should not have side effects on the remote system.

When discussing data sources, they are typically classified by the intended number of return objects or data. Singular data sources represent a one-to-one lookup or data operation. Plural data sources represent a one-to-many operation.

### Plural Data Sources

These data sources are intended to return zero, one, or many results, usually associated with a managed resource type. Typically results are a set unless ordering guarantees are provided by the remote system. These should be named with a plural suffix (e.g. `s` or `es`) and should not include any specific attribute in the naming (e.g. prefer `cloudflare_zones` instead of `cloudflare_zone_ids`).

### Singular Data Sources

These data sources are intended to return one result or an error. These should not include any specific attribute in the naming (e.g. prefer `cloudflare_zone` instead of `cloudflare_zone_id`).

[cloudflare-go]: https://github.com/cloudflare/cloudflare-go
[api.cloudflare.com]: https://api.cloudflare.com
[developers.cloudflare.com]: https://developers.cloudflare.com
[cf-terraforming]: https://github.com/cloudflare/cf-terraforming
