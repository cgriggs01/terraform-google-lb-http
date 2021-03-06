# Global HTTP Load Balancer Terraform Module

Modular Global HTTP Load Balancer for GCE using forwarding rules.

## Usage

```ruby
module "gce-lb-http" {
  source            = "github.com/GoogleCloudPlatform/terraform-google-lb-http"
  name              = "group-http-lb"
  target_tags       = ["${module.mig1.target_tags}", "${module.mig2.target_tags}"]
  backends          = {
    "0" = [
      { group = "${module.mig1.instance_group}" },
      { group = "${module.mig2.instance_group}" }
    ],
  }
  backend_params    = [
    # health check path, port name, port number, timeout seconds.
    "/,http,80,10"
  ]
}
```

### Input variables

- `region` (optional): Region for cloud resources. Default is `us-central1`.
- `network` (optional): Name of the network to create resources in. Default is `default`.
- `name` (required): Name for the forwarding rule and prefix for supporting resources.
- `target_tags` (required) List of target tags for health check firewall rule.
- `backends` (required): Map backend indices to list of [backend maps](https://www.terraform.io/docs/providers/google/r/compute_backend_service.html#backend).
- `backend_params` (required): Comma-separated encoded list of parameters in order: health check path, service port name, service port, backend timeout seconds.
- `create_url_map` (optional): Set to `false` if url_map variable is provided. Default is `true`.
- `url_map` (optional): The url_map resource to use. Default is to send all traffic to first backend.
- `ssl` (optional): Set to `true` to enable SSL support and create a second forwarding rule for HTTPS traffic, requires variables `private_key` and `certificate`. Default is `false`.
- `private_key` (optional): Content of the private SSL key. Required if ssl is `true`.
- `certificate` (optional): Content of the SSL certificate. Required if ssl is `true`.

### Output variables

- `backend_services`: The backend service resources.
- `external_ip`: The external IP assigned to the global fowarding rule.

## Resources created

**Figure 1.** *diagram of terraform resources*

![architecture diagram](./diagram.png)

- [`google_compute_global_forwarding_rule.http`](https://www.terraform.io/docs/providers/google/r/compute_global_forwarding_rule.html): The global HTTP forwarding rule.
- [`google_compute_global_forwarding_rule.https`](https://www.terraform.io/docs/providers/google/r/compute_global_forwarding_rule.html): The global HTTPS forwarding rule created when `ssl` is `true`.
- [`google_compute_target_http_proxy.default`](https://www.terraform.io/docs/providers/google/r/compute_target_http_proxy.html): The HTTP proxy resource that binds the url map. Created when input `ssl` is `false`.
- [`google_compute_target_https_proxy.default`](https://www.terraform.io/docs/providers/google/r/compute_target_https_proxy.html): The HTTPS proxy resource that binds the url map. Created when input `ssl` is `true`.
- [`google_compute_ssl_certificate.default`](https://www.terraform.io/docs/providers/google/r/compute_ssl_certificate.html): The certificate resource created when input `ssl` is `true`. 
- [`google_compute_url_map.default`](https://www.terraform.io/docs/providers/google/r/compute_url_map.html): The default URL map resource when input `url_map` is not provided.
- [`google_compute_backend_service.default.*`](https://www.terraform.io/docs/providers/google/r/compute_backend_service.html): The backend services created for each of the `backend_params` elements.
- [`google_compute_http_health_check.default.*`](https://www.terraform.io/docs/providers/google/r/compute_http_health_check.html): Health check resources create for each of the backend services.
- [`google_compute_firewall.default-hc`](https://www.terraform.io/docs/providers/google/r/compute_firewall.html): Firewall rule created for each of the backed services to alllow health checks to the instance group.