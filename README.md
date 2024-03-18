# zta-gcp
Creating a modular Terraform setup for deploying the specified GCP resources involves creating individual modules for each resource type (VPC, subnets, firewall rules, Compute Engine instances, Cloud Load Balancing with Cloud Armor, Cloud SQL, and VPC Service Controls). Then, you compose these modules together in a root Terraform configuration. This approach enhances modularity, reusability, and maintainability of your infrastructure as code.

Below is an outline on structuring the Terraform scripts for each step. Given the complexity and length, I'll provide a conceptual overview and examples for key parts, focusing on structure rather than complete code.

Step 1: Directory Structure
First, organize your Terraform configuration with a directory structure that separates each component into its own module:

modules/
├── cloud_sql
├── compute_instance
├── firewall
├── load_balancer
├── subnet
├── vpc
└── vpc_service_control
Each directory will contain at least two files: main.tf (for resource definitions) and variables.tf (for module input variables).

Step 2: Define Modules
VPC Module (modules/vpc/main.tf)
resource "google_compute_network" "vpc" {
  name                    = var.name
  auto_create_subnetworks = false
  project                 = var.project
  routing_mode            = "REGIONAL"
}
Define input variables in modules/vpc/variables.tf.

Subnet Module (modules/subnet/main.tf)
resource "google_compute_subnetwork" "subnet" {
  name          = var.name
  ip_cidr_range = var.ip_cidr_range
  region        = var.region
  network       = var.network_id
  project       = var.project
}
Define input variables in modules/subnet/variables.tf.

Firewall Module (modules/firewall/main.tf)
resource "google_compute_firewall" "firewall" {
  name    = var.name
  network = var.network

  allow {
    protocol = "tcp"
    ports    = ["22", "80", "443"]
  }

  source_ranges = ["0.0.0.0/0"]
}
Define input variables in modules/firewall/variables.tf.

Compute Instance Module
This would be more detailed, involving google_compute_instance resources, specifying the machine type, image, and network interfaces.

Cloud Load Balancing and Cloud Armor Module
Include resources like google_compute_forwarding_rule, google_compute_target_http_proxy, google_compute_url_map, and google_compute_backend_service, integrated with Cloud Armor for security.

Cloud SQL Module
Utilize google_sql_database_instance and google_sql_database resources, with a focus on high availability settings.

VPC Service Control Module
Define resources related to google_access_context_manager_service_perimeter to create security perimeters.

Step 3: Use Modules in Root Configuration
In your root Terraform configuration (main.tf), you'd reference these modules with the required inputs. For example:
module "vpc" {
  source  = "./modules/vpc"
  name    = "custom-vpc"
  project = "your-gcp-project-id"
}

module "subnet_us_central1_a" {
  source      = "./modules/subnet"
  name        = "subnet-us-central1-a"
  ip_cidr_range = "10.0.1.0/24"
  region      = "us-central1"
  network_id  = module.vpc.vpc_self_link
  project     = "your-gcp-project-id"
}

# Similar references for other modules...
This setup provides a starting point. Each module should be further developed to meet specific needs, using the Terraform documentation for each GCP resource as a guide. Remember, Terraform configurations must be adapted to fit the specific architecture and security requirements of your project.
