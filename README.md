# Cloudflare Terraforming

## Overview

`cf-terraforming` is a command line utility to facilitate terraforming your
existing Cloudflare resources. It does this by using your account credentials to
retrieve your configurations from the [Cloudflare API](https://api.cloudflare.com)
and converting them to Terraform configurations that can be used with the
[Terraform Cloudflare provider](https://registry.terraform.io/providers/cloudflare/cloudflare/latest).

This tool is ideal if you already have Cloudflare resources defined but want to
start managing them via Terraform, and don't want to spend the time to manually
write the Terraform configuration to describe them.

See also the [blog page](https://blog.cloudflare.com/cloudflares-partnership-with-hashicorp-and-bootstrapping-terraform-with-cf-terraforming/) for further details on installation and usage.

> NOTE: If you would like to export resources compatible with Terraform < 0.12.x,
> you will need to download an older release as this tool no longer supports it.

## Usage

```
Usage:
  cf-terraforming [command]

Available Commands:
  generate    Fetch resources from the Cloudflare API and generate the respective Terraform stanzas
  help        Help about any command
  import      Output `terraform import` compatible commands in order to import resources into state
  version     Print the version number of cf-terraforming

Flags:
  -a, --account string         Use specific account ID for commands
  -c, --config string          Path to configuration file (default is $HOME/.cf-terraforming.yaml)
  -e, --email string           API Email address associated with your account
  -h, --help                   Help for cf-terraforming
  -k, --key string             API Key generated on the 'My Profile' page. See: https://dash.cloudflare.com/profile
      --resource-type string   Which resource you wish to generate
  -t, --token string           API Token
  -v, --verbose                Specify verbose output (same as setting log level to debug)
  -z, --zone string            Limit the export to a single zone ID

Use "cf-terraforming [command] --help" for more information about a command.
```

## Authentication

Cloudflare supports two authentication methods to the API:
* API Token - gives access only to resources and permissions specified for that token (recommended)
* API key - gives access to everything your user profile has access to

Both can be retrieved on the [user profile page](https://dash.cloudflare.com/profile/api-tokens).

**A note on storing your credentials securely:** We recommend that you store
your Cloudflare credentials (API key, email, token) as environment variables as
demonstrated below.

```bash
# if using API Token
export CLOUDFLARE_API_TOKEN='Hzsq3Vub-7Y-hSTlAaLH3Jq_YfTUOCcgf22_Fs-j'

# if using API Key
export CLOUDFLARE_EMAIL='user@example.com'
export CLOUDFLARE_API_KEY='1150bed3f45247b99f7db9696fffa17cbx9'

# specify zone ID
export CLOUDFLARE_ZONE_ID='81b06ss3228f488fh84e5e993c2dc17'

# now call cf-terraforming, e.g.
cf-terraforming generate \
  --resource-type "cloudflare_record" \
  --zone $CLOUDFLARE_ZONE_ID
```

cf-terraforming supports the following environment variables:
* CLOUDFLARE_API_TOKEN - API Token based authentication
* CLOUDFLARE_EMAIL, CLOUDFLARE_API_KEY - API Key based authentication

Alternatively, if using a config file, then specify the inputs using the same
names the `flag` names. Example:

```
$ cat ~/.cf-terraforming.yaml
email: "email@domain.com"
key: "<key>"
#or
token: "<token>"
```

## Example usage

```bash
$ cf-terraforming generate \
  --zone $CLOUDFLARE_ZONE_ID \
  --resource-type "cloudflare_record"
```

will contact the Cloudflare API on your behalf and result in a valid Terraform
configuration representing the **resource** you requested:

```hcl
resource "cloudflare_record" "terraform_managed_resource" {
  name = "example.com"
  proxied = false
  ttl = 120
  type = "A"
  value = "198.51.100.4"
  zone_id = "0da42c8d2132a9ddaf714f9e7c920711"
}
```

## Prerequisites

* A Cloudflare account with resources defined (e.g. a few zones, some load
  balancers, spectrum applications, etc)
* A valid Cloudflare API key and sufficient permissions to access the resources
  you are requesting via the API
* An initialised Terraform directory (`terraform init` has run and providers installed). See the [provider documentation](https://registry.terraform.io/providers/cloudflare/cloudflare/latest/docs) if you have not yet setup the Terraform directory.

## Installation

If you use Homebrew on MacOS, you can run the following:

```bash
$ brew tap cloudflare/cloudflare
$ brew install --cask cloudflare/cloudflare/cf-terraforming
```

If you use another OS, you will need to download the release directly from
[GitHub Releases](https://github.com/cloudflare/cf-terraforming/releases).

## Importing with Terraform state

`cf-terraforming` will output the `terraform import` compatible commands for you
when you invoke the `import` command. This command assumes you have already ran
`cf-terraforming generate ...` to output your resources.

In the future we aim to automate this however for now, it is a manual step to
allow flexibility in directory structure.

```
$ cf-terraforming import \
  --resource-type "cloudflare_record" \
  --email $CLOUDFLARE_EMAIL \
  --key $CLOUDFLARE_API_KEY \
  --zone $CLOUDFLARE_ZONE_ID
```

## Supported Resources

The following resources can be used with both `generate` and `import`

Last updated Nov 2, 2021

| Resource | Supported |
|----------|-----------|
| [cloudflare_access_application](https://www.terraform.io/docs/providers/cloudflare/r/access_application) | ✅ |
| [cloudflare_access_group](https://www.terraform.io/docs/providers/cloudflare/r/access_group) | ❌ |
| [cloudflare_access_identity_provider](https://www.terraform.io/docs/providers/cloudflare/r/access_identity_provider) | ✅ |
| [cloudflare_access_mutual_tls_certificate](https://www.terraform.io/docs/providers/cloudflare/r/access_mutual_tls_certificate) | ❌ |
| [cloudflare_access_policy](https://www.terraform.io/docs/providers/cloudflare/r/access_policy) | ❌ |
| [cloudflare_access_rule](https://www.terraform.io/docs/providers/cloudflare/r/access_rule) | ✅ |
| [cloudflare_access_service_token](https://www.terraform.io/docs/providers/cloudflare/r/access_service_token) | ✅ |
| [cloudflare_account_member](https://www.terraform.io/docs/providers/cloudflare/r/account_member) | ✅ |
| [cloudflare_api_token](https://www.terraform.io/docs/providers/cloudflare/r/api_token) | ❌ |
| [cloudflare_argo](https://www.terraform.io/docs/providers/cloudflare/r/argo) | ✅ |
| [cloudflare_argo_tunnel](https://www.terraform.io/docs/providers/cloudflare/r/argo_tunnel) | ✅ |
| [cloudflare_authenticated_origin_pulls](https://www.terraform.io/docs/providers/cloudflare/r/authenticated_origin_pulls) | ❌ |
| [cloudflare_authenticated_origin_pulls_certificate](https://www.terraform.io/docs/providers/cloudflare/r/authenticated_origin_pulls_certificate) | ❌ |
| [cloudflare_byo_ip_prefix](https://www.terraform.io/docs/providers/cloudflare/r/byo_ip_prefix) | ✅ |
| [cloudflare_certificate_pack]Ï(https://www.terraform.io/docs/providers/cloudflare/r/certificate_pack) | ✅ |
| [cloudflare_custom_hostname](https://www.terraform.io/docs/providers/cloudflare/r/custom_hostname) | ✅ |
| [cloudflare_custom_hostname_fallback_origin](https://www.terraform.io/docs/providers/cloudflare/r/custom_hostname_fallback_origin) | ✅ |
| [cloudflare_custom_pages](https://www.terraform.io/docs/providers/cloudflare/r/custom_pages) | ✅ |
| [cloudflare_custom_ssl](https://www.terraform.io/docs/providers/cloudflare/r/custom_ssl) | ❌ |
| [cloudflare_filter](https://www.terraform.io/docs/providers/cloudflare/r/filter) | ✅ |
| [cloudflare_firewall_rule](https://www.terraform.io/docs/providers/cloudflare/r/firewall_rule) | ✅ |
| [cloudflare_healthcheck](https://www.terraform.io/docs/providers/cloudflare/r/healthcheck) | ✅ |
| [cloudflare_ip_list](https://www.terraform.io/docs/providers/cloudflare/r/ip_list) | ❌ |
| [cloudflare_load_balancer](https://www.terraform.io/docs/providers/cloudflare/r/load_balancer) | ✅ |
| [cloudflare_load_balancer_monitor](https://www.terraform.io/docs/providers/cloudflare/r/load_balancer_monitor) | ✅ |
| [cloudflare_load_balancer_pool](https://www.terraform.io/docs/providers/cloudflare/r/load_balancer_pool) | ✅ |
| [cloudflare_logpull_retention](https://www.terraform.io/docs/providers/cloudflare/r/logpull_retention) | ❌ |
| [cloudflare_logpush_job](https://www.terraform.io/docs/providers/cloudflare/r/logpush_job) | ✅ |
| [cloudflare_logpush_ownership_challenge](https://www.terraform.io/docs/providers/cloudflare/r/logpush_ownership_challenge) | ❌ |
| [cloudflare_magic_firewall_ruleset](https://www.terraform.io/docs/providers/cloudflare/r/magic_firewall_ruleset) | ❌ |
| [cloudflare_origin_ca_certificate](https://www.terraform.io/docs/providers/cloudflare/r/origin_ca_certificate) | ✅ |
| [cloudflare_page_rule](https://www.terraform.io/docs/providers/cloudflare/r/page_rule) | ✅ |
| [cloudflare_rate_limit](https://www.terraform.io/docs/providers/cloudflare/r/rate_limit) | ✅ |
| [cloudflare_record](https://www.terraform.io/docs/providers/cloudflare/r/record) | ✅ |
| [cloudflare_ruleset](https://www.terraform.io/docs/providers/cloudflare/r/ruleset) | ✅ |
| [cloudflare_spectrum_application](https://www.terraform.io/docs/providers/cloudflare/r/spectrum_application) | ✅ |
| [cloudflare_waf_group](https://www.terraform.io/docs/providers/cloudflare/r/waf_group) | ❌ |
| [cloudflare_waf_override](https://www.terraform.io/docs/providers/cloudflare/r/waf_override) | ✅ |
| [cloudflare_waf_package](https://www.terraform.io/docs/providers/cloudflare/r/waf_package) | ❌ |
| [cloudflare_waf_rule](https://www.terraform.io/docs/providers/cloudflare/r/waf_rule) | ❌ |
| [cloudflare_worker_cron_trigger](https://www.terraform.io/docs/providers/cloudflare/r/worker_cron_trigger) | ❌ |
| [cloudflare_worker_route](https://www.terraform.io/docs/providers/cloudflare/r/worker_route) | ✅ |
| [cloudflare_worker_script](https://www.terraform.io/docs/providers/cloudflare/r/worker_script) | ❌ |
| [cloudflare_workers_kv](https://www.terraform.io/docs/providers/cloudflare/r/workers_kv) | ❌ |
| [cloudflare_workers_kv_namespace](https://www.terraform.io/docs/providers/cloudflare/r/workers_kv_namespace) | ✅ |
| [cloudflare_zone](https://www.terraform.io/docs/providers/cloudflare/r/zone) | ✅ |
| [cloudflare_zone_dnssec](https://www.terraform.io/docs/providers/cloudflare/r/zone_dnssec) | ❌ |
| [cloudflare_zone_lockdown](https://www.terraform.io/docs/providers/cloudflare/r/zone_lockdown) | ✅ |
| [cloudflare_zone_settings_override](https://www.terraform.io/docs/providers/cloudflare/r/zone_settings_override) | ✅ |

## Testing

To ensure changes don't introduce regressions this tool uses an automated test
suite consisting of HTTP mocks via go-vcr and Terraform configuration files to
assert against. The premise is that we mock the HTTP responses from the
Cloudflare API to ensure we don't need to create and delete real resources to
test. The Terraform files then allow us to build what the resource structure is
expected to look like and once the tool parses the API response, we can compare
that to the static file.
