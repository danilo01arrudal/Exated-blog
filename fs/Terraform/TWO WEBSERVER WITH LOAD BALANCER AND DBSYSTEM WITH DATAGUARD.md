##   TERRAFORM WITH OCI DEPLOY TWO WEBSERVER WITH LOAD BALANCER AND DBSYSTEM WITH DATAGUARD
### PLAN, APPLY AND DESTROY TWO SIMPLE WEBSERVICE ON ORACLE CLOUD OCI USING TERRAFORM WITH LOAD BALANCER AND DBSYSTEM WITH DATAGUARD

###### File content - vcn.tf
		resource "oci_core_virtual_network" "ExatedVCN" {
		cidr_block = var.VCN-CIDR
		dns_label = "ExatedVCN"
		compartment_id = oci_identity_compartment.ExatedCompartment.id
		display_name = "ExatedVCN"
		}
		
		# Gets a list of Availability Domains
		data "oci_identity_availability_domains" "ADs" {
		compartment_id = var.tenancy_ocid
		}
		
		# Gets the Id of a specific OS Images
		data "oci_core_images" "OSImageLocal" {
		#Required
		compartment_id = var.compartment_ocid
		display_name   = var.OsImage
		}
		
###### File content - variables_UPDATED.tf
		variable "tenancy_ocid" {}
		variable "user_ocid" {}
		variable "fingerprint" {}
		variable "private_key_path" {}
		variable "compartment_ocid" {}
		variable "region" {}
		variable "private_key_oci" {}
		variable "public_key_oci" {}
		
		variable "VCN-CIDR" {
		default = "10.0.0.0/16"
		}
		
		variable "VCNname" {
		default = "ExatedVCN"
		}
		
		variable "websubnet-CIDR" {
		default = "10.0.1.0/24"
		}
		
		variable "lbsubnet-CIDR" {
		default = "10.0.2.0/24"
		}
		
		variable "bastionsubnet-CIDR" {
		default = "10.0.3.0/24"
		}
		
		variable "fsssubnet-CIDR" {
		default = "10.0.5.0/24"
		}
		
		
		variable "Shapes" {
		default = ["VM.Standard.E2.1","VM.Standard.E2.1.Micro","VM.Standard2.1","VM.Standard.E2.1","VM.Standard.E2.2"]
		}
		
		variable "OsImage" {
		default = "Oracle-Linux-7.8-2020.05.26-0"
		}
		
		variable "webservice_ports" {
		default = ["80","443"]
		}
		
		variable "bastion_ports" {
		default = ["22"]
		}
		
		variable "sqlnet_ports" {
		default = ["1521"]
		}
		
		variable "fss_ingress_tcp_ports" {
		default = ["111","2048","2049","2050"]
		}
		
		variable "fss_ingress_udp_ports" {
		default = ["111","2048"]
		}
		
		variable "fss_egress_tcp_ports" {
		default = ["111","2048","2049","2050"]
		}
		
		variable "fss_egress_udp_ports" {
		default = ["111"]
		}
		
		# DBSystem specific 
		variable "DBNodeShape" {
			default = "VM.Standard2.1"
		}
		
		# DBStandbySystem specific 
		variable "DBStandbyNodeShape" {
			default = "VM.Standard2.1"
		}
		
		variable "CPUCoreCount" {
			default = "1"
		}
		
		variable "DBEdition" {
			default = "ENTERPRISE_EDITION"
		}
		
		variable "DBAdminPassword" {
			default = "BEstrO0ng_#11"
		}
		
		variable "DBName" {
			default = "EXATEDDB"
		}
		
		variable "DBVersion" {
			default = "12.1.0.2"
		}
		
		variable "DBDisplayName" {
			default = "EXATEDDB"
		}
		
		variable "DBHomeDisplayName" {
			default = "EXATEDDBHome"
		}
		
		variable "DBDiskRedundancy" {
			default = "HIGH"
		}
		
		variable "DBSystemDisplayName" {
			default = "ExatedDBSystem"
		}
		
		variable "DBStandbySystemDisplayName" {
			default = "ExatedDBStandbySystem"
		}
		
		variable "DBNodeDomainName" {
			default = "ExatedN4.ExatedVCN.oraclevcn.com"
		}
		
		variable "DBNodeHostName" {
			default = "EXATEDDBpri"
		}
		
		variable "DBStandbyNodeHostName" {
			default = "EXATEDDBstb"
		}
		
		variable "HostUserName" {
			default = "opc"
		}
		
		variable "NCharacterSet" {
		default = "AL16UTF16"
		}
		
		variable "CharacterSet" {
		default = "AL32UTF8"
		}
		
		variable "DBWorkload" {
		default = "OLTP"
		}
		
		variable "PDBName" {
		default = "mysla"
		}
		
		variable "DataStorageSizeInGB" {
		default = "256"
		}
		
		variable "LicenseModel" {
		default = "LICENSE_INCLUDED"
		}
		
		variable "NodeCount" {
		default = "1"
		}
		
###### File content - subnet1.tf
		resource "oci_core_subnet" "ExatedLBSubnet" {
		cidr_block = "10.0.2.0/24"
		display_name = "ExatedLBSubnet"
		dns_label = "ExatedN1"
		compartment_id = oci_identity_compartment.ExatedCompartment.id
		vcn_id = oci_core_virtual_network.ExatedVCN.id
		route_table_id = oci_core_route_table.ExatedRouteTableViaIGW.id
		dhcp_options_id = oci_core_dhcp_options.ExatedDhcpOptions1.id
		#  security_list_ids = [oci_core_security_list.ExatedWebSecurityList.id]
		}
		
###### File content - subnet2.tf
		resource "oci_core_subnet" "ExatedWebSubnet" {
		cidr_block = "10.0.1.0/24"
		display_name = "ExatedWebSubnet"
		dns_label = "ExatedN2"
		compartment_id = oci_identity_compartment.ExatedCompartment.id
		vcn_id = oci_core_virtual_network.ExatedVCN.id
		route_table_id = oci_core_route_table.ExatedRouteTableViaNAT.id
		dhcp_options_id = oci_core_dhcp_options.ExatedDhcpOptions1.id
		#  security_list_ids = [oci_core_security_list.ExatedWebSecurityList.id,oci_core_security_list.ExatedSSHSecurityList.id]
		}
		
###### File content - subnet3.tf
		resource "oci_core_subnet" "ExatedBastionSubnet" {
		cidr_block = "10.0.3.0/24"
		display_name = "ExatedBastionSubnet"
		dns_label = "ExatedN3"
		compartment_id = oci_identity_compartment.ExatedCompartment.id
		vcn_id = oci_core_virtual_network.ExatedVCN.id
		route_table_id = oci_core_route_table.ExatedRouteTableViaIGW.id
		dhcp_options_id = oci_core_dhcp_options.ExatedDhcpOptions1.id
		#  security_list_ids = [oci_core_security_list.ExatedSSHSecurityList.id]
		}
		
###### File content - subnet4_NEW.tf
		resource "oci_core_subnet" "ExatedDBSubnet" {
		cidr_block = "10.0.4.0/24"
		display_name = "ExatedDBSubnet"
		dns_label = "ExatedN4"
		prohibit_public_ip_on_vnic = true
		compartment_id = oci_identity_compartment.ExatedCompartment.id
		vcn_id = oci_core_virtual_network.ExatedVCN.id
		route_table_id = oci_core_route_table.ExatedRouteTableViaNAT.id
		dhcp_options_id = oci_core_dhcp_options.ExatedDhcpOptions1.id
		#  security_list_ids = [oci_core_security_list.ExatedSSHSecurityList.id,oci_core_security_list.ExatedSQLNetSecurityList.id]
		}
		
###### File content - security_list1_REMOVED.tf
		/*resource "oci_core_security_list" "ExatedWebSecurityList" {
			compartment_id = oci_identity_compartment.ExatedCompartment.id
			display_name = "ExatedWebSecurityList"
			vcn_id = oci_core_virtual_network.ExatedVCN.id
			
			egress_security_rules {
				protocol = "6"
				destination = "0.0.0.0/0"
			}
			
			dynamic "ingress_security_rules" {
			for_each = var.webservice_ports
			content {
				protocol = "6"
				source = "0.0.0.0/0"
				tcp_options {
					max = ingress_security_rules.value
					min = ingress_security_rules.value
					}
				}
			}
		
			ingress_security_rules {
				protocol = "6"
				source = var.VCN-CIDR
			}
		}
		*/
		
###### File content - security_list2_REMOVED.tf
		/*resource "oci_core_security_list" "ExatedSSHSecurityList" {
			compartment_id = oci_identity_compartment.ExatedCompartment.id
			display_name = "ExatedSSHSecurityList"
			vcn_id = oci_core_virtual_network.ExatedVCN.id
			
			egress_security_rules {
				protocol = "6"
				destination = "0.0.0.0/0"
			}
			
			dynamic "ingress_security_rules" {
			for_each = var.bastion_ports
			content {
				protocol = "6"
				source = "0.0.0.0/0"
				tcp_options {
					max = ingress_security_rules.value
					min = ingress_security_rules.value
					}
				}
			}
		
			ingress_security_rules {
				protocol = "6"
				source = var.VCN-CIDR
			}
		}
		*/
		
###### File content - security_list3_REMOVED.tf
		/*resource "oci_core_security_list" "ExatedSQLNetSecurityList" {
			compartment_id = oci_identity_compartment.ExatedCompartment.id
			display_name = "Foggy Kitchen SQLNet Security List"
			vcn_id = oci_core_virtual_network.ExatedVCN.id
			
			egress_security_rules {
				protocol = "6"
				destination = "0.0.0.0/0"
			}
			
			dynamic "ingress_security_rules" {
			for_each = var.sqlnet_ports
			content {
				protocol = "6"
				source = "0.0.0.0/0"
				tcp_options {
					max = ingress_security_rules.value
					min = ingress_security_rules.value
					}
				}
			}
		
			ingress_security_rules {
				protocol = "6"
				source = var.VCN-CIDR
			}
		}
		*/
		
###### File content - security_group_rule_NEW.tf
		resource "oci_core_network_security_group_security_rule" "ExatedFSSSecurityIngressTCPGroupRules" {
			for_each = toset(var.fss_ingress_tcp_ports)
		
			network_security_group_id = oci_core_network_security_group.ExatedFSSSecurityGroup.id
			direction = "INGRESS"
			protocol = "6"
			source = var.websubnet-CIDR
			source_type = "CIDR_BLOCK"
			tcp_options {
				destination_port_range {
					max = each.value
					min = each.value
				}
			}
		}
		
		resource "oci_core_network_security_group_security_rule" "ExatedFSSSecurityIngressUDPGroupRules" {
			for_each = toset(var.fss_ingress_udp_ports)
		
			network_security_group_id = oci_core_network_security_group.ExatedFSSSecurityGroup.id
			direction = "INGRESS"
			protocol = "17"
			source = var.websubnet-CIDR
			source_type = "CIDR_BLOCK"
			udp_options {
				destination_port_range {
					max = each.value
					min = each.value
				}
			}
		}
		
		resource "oci_core_network_security_group_security_rule" "ExatedFSSSecurityEgressTCPGroupRules" {
			for_each = toset(var.fss_egress_tcp_ports)
		
			network_security_group_id = oci_core_network_security_group.ExatedFSSSecurityGroup.id
			direction = "EGRESS"
			protocol = "6"
			destination = var.websubnet-CIDR
			destination_type = "CIDR_BLOCK"
			tcp_options {
				destination_port_range {
					max = each.value
					min = each.value
				}
			}
		}
		
		resource "oci_core_network_security_group_security_rule" "ExatedFSSSecurityEgressUDPGroupRules" {
			for_each = toset(var.fss_egress_udp_ports)
		
			network_security_group_id = oci_core_network_security_group.ExatedFSSSecurityGroup.id
			direction = "EGRESS"
			protocol = "17"
			destination = var.websubnet-CIDR
			destination_type = "CIDR_BLOCK"
			udp_options {
				destination_port_range {
					max = each.value
					min = each.value
				}
			}
		}
		
		
		resource "oci_core_network_security_group_security_rule" "ExatedWebSecurityEgressGroupRule" {
			network_security_group_id = oci_core_network_security_group.ExatedWebSecurityGroup.id
			direction = "EGRESS"
			protocol = "6"
			destination = "0.0.0.0/0"
			destination_type = "CIDR_BLOCK"
		}
		
		resource "oci_core_network_security_group_security_rule" "ExatedWebSecurityIngressGroupRules" {
			for_each = toset(var.webservice_ports)
		
			network_security_group_id = oci_core_network_security_group.ExatedWebSecurityGroup.id
			direction = "INGRESS"
			protocol = "6"
			source = "0.0.0.0/0"
			source_type = "CIDR_BLOCK"
			tcp_options {
				destination_port_range {
					max = each.value
					min = each.value
				}
			}
		}
		
		resource "oci_core_network_security_group_security_rule" "ExatedSSHSecurityEgressGroupRule" {
			network_security_group_id = oci_core_network_security_group.ExatedSSHSecurityGroup.id
			direction = "EGRESS"
			protocol = "6"
			destination = "0.0.0.0/0"
			destination_type = "CIDR_BLOCK"
		}
		
		resource "oci_core_network_security_group_security_rule" "ExatedSSHSecurityIngressGroupRules" {
			for_each = toset(var.bastion_ports)
		
			network_security_group_id = oci_core_network_security_group.ExatedSSHSecurityGroup.id
			direction = "INGRESS"
			protocol = "6"
			source = "0.0.0.0/0"
			source_type = "CIDR_BLOCK"
			tcp_options {
				destination_port_range {
					max = each.value
					min = each.value
				}
			}
		}
		
		
		resource "oci_core_network_security_group_security_rule" "ExatedDBSystemSecurityEgressGroupRule" {
			network_security_group_id = oci_core_network_security_group.ExatedDBSystemSecurityGroup.id
			direction = "EGRESS"
			protocol = "6"
			destination = "0.0.0.0/0"
			destination_type = "CIDR_BLOCK"
		}
		
		resource "oci_core_network_security_group_security_rule" "ExatedDBSystemSecurityIngressGroupRules" {
			for_each = toset(var.sqlnet_ports)
		
			network_security_group_id = oci_core_network_security_group.ExatedDBSystemSecurityGroup.id
			direction = "INGRESS"
			protocol = "6"
			source = "0.0.0.0/0"
			source_type = "CIDR_BLOCK"
			tcp_options {
				destination_port_range {
					max = each.value
					min = each.value
				}
			}
		}
		
###### File content - security_group_NEW.tf
		resource "oci_core_network_security_group" "ExatedWebSecurityGroup" {
			compartment_id = oci_identity_compartment.ExatedCompartment.id
			display_name = "ExatedWebSecurityGroup"
			vcn_id = oci_core_virtual_network.ExatedVCN.id
		}
		
		resource "oci_core_network_security_group" "ExatedSSHSecurityGroup" {
			compartment_id = oci_identity_compartment.ExatedCompartment.id
			display_name = "ExatedSSHSecurityGroup"
			vcn_id = oci_core_virtual_network.ExatedVCN.id
		}
		
		resource "oci_core_network_security_group" "ExatedFSSSecurityGroup" {
			compartment_id = oci_identity_compartment.ExatedCompartment.id
			display_name = "ExatedFSSSecurityGroup"
			vcn_id = oci_core_virtual_network.ExatedVCN.id
		}
		
		resource "oci_core_network_security_group" "ExatedDBSystemSecurityGroup" {
			compartment_id = oci_identity_compartment.ExatedCompartment.id
			display_name = "ExatedDBSystemSecurityGroup"
			vcn_id = oci_core_virtual_network.ExatedVCN.id
		}
		
###### File content - route1.tf
		resource "oci_core_route_table" "ExatedRouteTableViaIGW" {
			compartment_id = oci_identity_compartment.ExatedCompartment.id
			vcn_id = oci_core_virtual_network.ExatedVCN.id
			display_name = "ExatedRouteTableViaIGW"
			route_rules {
				destination = "0.0.0.0/0"
				destination_type  = "CIDR_BLOCK"
				network_entity_id = oci_core_internet_gateway.ExatedInternetGateway.id
			}
		}
		
###### File content - route2.tf
		resource "oci_core_route_table" "ExatedRouteTableViaNAT" {
			compartment_id = oci_identity_compartment.ExatedCompartment.id
			vcn_id = oci_core_virtual_network.ExatedVCN.id
			display_name = "ExatedRouteTableViaNAT"
			route_rules {
				destination = "0.0.0.0/0"
				destination_type  = "CIDR_BLOCK"
				network_entity_id = oci_core_nat_gateway.ExatedNATGateway.id
			}
		}
		
###### File content - natgateway.tf
		resource "oci_core_nat_gateway" "ExatedNATGateway" {
			compartment_id = oci_identity_compartment.ExatedCompartment.id
			display_name = "ExatedNATGateway"
			vcn_id = oci_core_virtual_network.ExatedVCN.id
		}
		
###### File content - loadbalancer.tf
		resource "oci_load_balancer" "ExatedPublicLoadBalancer" {
		shape          = "100Mbps"
		compartment_id = oci_identity_compartment.ExatedCompartment.id 
		subnet_ids     = [
			oci_core_subnet.ExatedLBSubnet.id
		]
		display_name   = "ExatedPublicLoadBalancer"
		network_security_group_ids = [oci_core_network_security_group.ExatedWebSecurityGroup.id]
		}
		
		resource "oci_load_balancer_backendset" "ExatedPublicLoadBalancerBackendset" {
		name             = "ExatedPublicLBBackendset"
		load_balancer_id = oci_load_balancer.ExatedPublicLoadBalancer.id
		policy           = "ROUND_ROBIN"
		
		health_checker {
			port     = "80"
			protocol = "HTTP"
			response_body_regex = ".*"
			url_path = "/shared/"
			interval_ms = "3000"
		}
		}
		
		
		resource "oci_load_balancer_listener" "ExatedPublicLoadBalancerListener" {
		load_balancer_id         = oci_load_balancer.ExatedPublicLoadBalancer.id
		name                     = "ExatedPublicLoadBalancerListener"
		default_backend_set_name = oci_load_balancer_backendset.ExatedPublicLoadBalancerBackendset.name
		port                     = 80
		protocol                 = "HTTP"
		}
		
		
		resource "oci_load_balancer_backend" "ExatedPublicLoadBalancerBackend1" {
		load_balancer_id = oci_load_balancer.ExatedPublicLoadBalancer.id
		backendset_name  = oci_load_balancer_backendset.ExatedPublicLoadBalancerBackendset.name
		ip_address       = oci_core_instance.ExatedWebserver1.private_ip
		port             = 80 
		backup           = false
		drain            = false
		offline          = false
		weight           = 1
		}
		
		resource "oci_load_balancer_backend" "ExatedPublicLoadBalancerBackend2" {
		load_balancer_id = oci_load_balancer.ExatedPublicLoadBalancer.id
		backendset_name  = oci_load_balancer_backendset.ExatedPublicLoadBalancerBackendset.name
		ip_address       = oci_core_instance.ExatedWebserver2.private_ip
		port             = 80
		backup           = false
		drain            = false
		offline          = false
		weight           = 1
		}
		
		
		output "ExatedPublicLoadBalancer_Public_IP" {
		value = [oci_load_balancer.ExatedPublicLoadBalancer.ip_addresses]
		}
		
###### File content - internet_gateway.tf
		resource "oci_core_internet_gateway" "ExatedInternetGateway" {
			compartment_id = oci_identity_compartment.ExatedCompartment.id
			display_name = "ExatedInternetGateway"
			vcn_id = oci_core_virtual_network.ExatedVCN.id
		}
		
###### File content - httpd_on_webserver1.tf
		resource "null_resource" "ExatedWebserver1HTTPD" {
		depends_on = [oci_core_instance.ExatedWebserver1,oci_core_instance.ExatedBastionServer,null_resource.ExatedWebserver1SharedFilesystem]
		provisioner "remote-exec" {
				connection {
						type     = "ssh"
						user     = "opc"
						host     = data.oci_core_vnic.ExatedWebserver1_VNIC1.private_ip_address
						private_key = file(var.private_key_oci)
						script_path = "/home/opc/myssh.sh"
						agent = false
						timeout = "10m"
						bastion_host = data.oci_core_vnic.ExatedBastionServer_VNIC1.public_ip_address
						bastion_port = "22"
						bastion_user = "opc"
						bastion_private_key = file(var.private_key_oci)
				}
		inline = ["echo '== 1. Installing HTTPD package with yum'",
					"sudo -u root yum -y -q install httpd",
		
					"echo '== 2. Creating /sharedfs/index.html'",
					"sudo -u root touch /sharedfs/index.html", 
					"sudo /bin/su -c \"echo 'Welcome to Exatedwebserver.com! These are both WEBSERVERS under LB umbrella with shared index.html ...' > /sharedfs/index.html\"",
					
					"echo '== 3. Adding Alias and Directory sharedfs to /etc/httpd/conf/httpd.conf'",
					"sudo /bin/su -c \"echo 'Alias /shared/ /sharedfs/' >> /etc/httpd/conf/httpd.conf\"",
					"sudo /bin/su -c \"echo '<Directory /sharedfs>' >> /etc/httpd/conf/httpd.conf\"",
					"sudo /bin/su -c \"echo 'AllowOverride All' >> /etc/httpd/conf/httpd.conf\"",
					"sudo /bin/su -c \"echo 'Require all granted' >> /etc/httpd/conf/httpd.conf\"",
					"sudo /bin/su -c \"echo '</Directory>' >> /etc/httpd/conf/httpd.conf\"",
		
					"echo '== 4. Disabling SELinux'",
					"sudo -u root setenforce 0",
		
					"echo '== 5. Disabling firewall and starting HTTPD service'",
					"sudo -u root service firewalld stop",
					"sudo -u root service httpd start"]
		}
		}
		
###### File content - httpd_on_webserver2.tf
		resource "null_resource" "ExatedWebserver2HTTPD" {
		depends_on = [oci_core_instance.ExatedWebserver2,oci_core_instance.ExatedBastionServer,null_resource.ExatedWebserver2SharedFilesystem]
		provisioner "remote-exec" {
				connection {
						type     = "ssh"
						user     = "opc"
						host     = data.oci_core_vnic.ExatedWebserver2_VNIC1.private_ip_address
						private_key = file(var.private_key_oci)
						script_path = "/home/opc/myssh.sh"
						agent = false
						timeout = "10m"
						bastion_host = data.oci_core_vnic.ExatedBastionServer_VNIC1.public_ip_address
						bastion_port = "22"
						bastion_user = "opc"
						bastion_private_key = file(var.private_key_oci)
				}
		inline = ["echo '== 1. Installing HTTPD package with yum'",
					"sudo -u root yum -y -q install httpd",
					
					"echo '== 2. Adding Alias and Directory sharedfs to /etc/httpd/conf/httpd.conf'",
					"sudo /bin/su -c \"echo 'Alias /shared/ /sharedfs/' >> /etc/httpd/conf/httpd.conf\"",
					"sudo /bin/su -c \"echo '<Directory /sharedfs>' >> /etc/httpd/conf/httpd.conf\"",
					"sudo /bin/su -c \"echo 'AllowOverride All' >> /etc/httpd/conf/httpd.conf\"",
					"sudo /bin/su -c \"echo 'Require all granted' >> /etc/httpd/conf/httpd.conf\"",
					"sudo /bin/su -c \"echo '</Directory>' >> /etc/httpd/conf/httpd.conf\"",
		
					"echo '== 3. Disabling SELinux'",
					"sudo -u root setenforce 0",
		
					"echo '== 4. Disabling firewall and starting HTTPD service'",
					"sudo -u root service firewalld stop",
					"sudo -u root service httpd start"]
		}
		}
		
###### File content - compartment.tf
		resource "oci_identity_compartment" "ExatedCompartment" {
		name = "ExatedCompartment"
		description = "Exated Compartment"
		compartment_id = var.compartment_ocid
		}
		
###### File content - dhcp_options.tf
		resource "oci_core_dhcp_options" "ExatedDhcpOptions1" {
		compartment_id = oci_identity_compartment.ExatedCompartment.id
		vcn_id = oci_core_virtual_network.ExatedVCN.id
		display_name = "ExatedDHCPOptions1"
		
		// required
		options {
			type = "DomainNameServer"
			server_type = "VcnLocalPlusInternet"
		}
		
		// optional
		options {
			type = "SearchDomain"
			search_domain_names = [ "Exatedwebserver.com" ]
		}
		}
		
###### File content - provider.tf
		provider "oci" {
		version          = ">= 4.19.0" 
		tenancy_ocid     = var.tenancy_ocid
		user_ocid        = var.user_ocid
		fingerprint      = var.fingerprint
		private_key_path = var.private_key_path
		region           = var.region
		}
		
###### File content - shared_filesystem.tf
		resource "oci_file_storage_mount_target" "ExatedMountTarget" {
		availability_domain = lookup(data.oci_identity_availability_domains.ADs.availability_domains[0], "name")
		compartment_id = oci_identity_compartment.ExatedCompartment.id
		subnet_id = oci_core_subnet.ExatedWebSubnet.id
		ip_address = "10.0.1.25"
		display_name = "ExatedMountTarget"
		nsg_ids = [oci_core_network_security_group.ExatedFSSSecurityGroup.id]
		}
		
		resource "oci_file_storage_export_set" "ExatedExportset" {
		mount_target_id = oci_file_storage_mount_target.ExatedMountTarget.id
		display_name = "ExatedExportset"
		}
		
		resource "oci_file_storage_file_system" "ExatedFilesystem" {
		availability_domain = lookup(data.oci_identity_availability_domains.ADs.availability_domains[0], "name")
		compartment_id = oci_identity_compartment.ExatedCompartment.id
		display_name = "ExatedFilesystem"
		}
		
		resource "oci_file_storage_export" "ExatedExport" {
		export_set_id = oci_file_storage_mount_target.ExatedMountTarget.export_set_id
		file_system_id = oci_file_storage_file_system.ExatedFilesystem.id
		path = "/sharedfs"
		}
		
		
		resource "null_resource" "ExatedWebserver1SharedFilesystem" {
		depends_on = [oci_core_instance.ExatedWebserver1,oci_core_instance.ExatedBastionServer,oci_file_storage_export.ExatedExport]
		
		provisioner "remote-exec" {
		connection {
						type                = "ssh"
						user                = "opc"
						host                = data.oci_core_vnic.ExatedWebserver1_VNIC1.private_ip_address
						private_key         = file(var.private_key_oci)
						script_path         = "/home/opc/myssh.sh"
						agent               = false
						timeout             = "10m"
						bastion_host        = data.oci_core_vnic.ExatedBastionServer_VNIC1.public_ip_address
						bastion_port        = "22"
						bastion_user        = "opc"
						bastion_private_key = file(var.private_key_oci)
				}
		inline = [
					"sudo /bin/su -c \"yum install -y -q nfs-utils\"",
					"sudo /bin/su -c \"mkdir -p /sharedfs\"",
					"sudo /bin/su -c \"echo '10.0.1.25:/sharedfs /sharedfs nfs rsize=8192,wsize=8192,timeo=14,intr 0 0' >> /etc/fstab\"",
					"sudo /bin/su -c \"mount /sharedfs\""
					]
		}
		
		}
		
		resource "null_resource" "ExatedWebserver2SharedFilesystem" {
		depends_on = [oci_core_instance.ExatedWebserver2,oci_core_instance.ExatedBastionServer,oci_file_storage_export.ExatedExport]
		
		provisioner "remote-exec" {
		connection {
						type                = "ssh"
						user                = "opc"
						host                = data.oci_core_vnic.ExatedWebserver2_VNIC1.private_ip_address
						private_key         = file(var.private_key_oci)
						script_path         = "/home/opc/myssh.sh"
						agent               = false
						timeout             = "10m"
						bastion_host        = data.oci_core_vnic.ExatedBastionServer_VNIC1.public_ip_address
						bastion_port        = "22"
						bastion_user        = "opc"
						bastion_private_key = file(var.private_key_oci)
				}
		inline = [
					"sudo /bin/su -c \"yum install -y -q nfs-utils\"",
					"sudo /bin/su -c \"mkdir -p /sharedfs\"",
					"sudo /bin/su -c \"echo '10.0.1.25:/sharedfs /sharedfs nfs rsize=8192,wsize=8192,timeo=14,intr 0 0' >> /etc/fstab\"",
					"sudo /bin/su -c \"mount /sharedfs\""
					]
		}
		
		}
		
###### File content - webserver1_block_volumes.tf
		resource "oci_core_volume" "ExatedWebserver1BlockVolume100G" {
		availability_domain = lookup(data.oci_identity_availability_domains.ADs.availability_domains[0], "name")
		compartment_id = oci_identity_compartment.ExatedCompartment.id
		display_name = "ExatedWebserver1 BlockVolume 100G"
		size_in_gbs = "100"
		}
		
		resource "oci_core_volume_attachment" "ExatedWebserver1BlockVolume100G_attach" {
			attachment_type = "iscsi"
			compartment_id = oci_identity_compartment.ExatedCompartment.id
			instance_id = oci_core_instance.ExatedWebserver1.id
			volume_id = oci_core_volume.ExatedWebserver1BlockVolume100G.id
		}
		
		
		resource "null_resource" "ExatedWebserver1_oci_iscsi_attach" {
		depends_on = [oci_core_volume_attachment.ExatedWebserver1BlockVolume100G_attach]
		
		provisioner "remote-exec" {
			connection {
						type     = "ssh"
						user     = "opc"
						host     = data.oci_core_vnic.ExatedWebserver1_VNIC1.private_ip_address
						private_key = file(var.private_key_oci)
						script_path = "/home/opc/myssh.sh"
						agent = false
						timeout = "10m"
						bastion_host = data.oci_core_vnic.ExatedBastionServer_VNIC1.public_ip_address
						bastion_port = "22"
						bastion_user = "opc"
						bastion_private_key = file(var.private_key_oci)
				}
			inline = ["sudo /bin/su -c \"rm -Rf /home/opc/iscsiattach.sh\""]
		}
		
		provisioner "file" {
			connection {
						type     = "ssh"
						user     = "opc"
						host     = data.oci_core_vnic.ExatedWebserver1_VNIC1.private_ip_address
						private_key = file(var.private_key_oci)
						script_path = "/home/opc/myssh.sh"
						agent = false
						timeout = "10m"
						bastion_host = data.oci_core_vnic.ExatedBastionServer_VNIC1.public_ip_address
						bastion_port = "22"
						bastion_user = "opc"
						bastion_private_key = file(var.private_key_oci)
				}
			source     = "iscsiattach.sh"
			destination = "/home/opc/iscsiattach.sh"
		}
		
		provisioner "remote-exec" {
					connection {
						type     = "ssh"
						user     = "opc"
						host     = data.oci_core_vnic.ExatedWebserver1_VNIC1.private_ip_address
						private_key = file(var.private_key_oci)
						script_path = "/home/opc/myssh.sh"
						agent = false
						timeout = "10m"
						bastion_host = data.oci_core_vnic.ExatedBastionServer_VNIC1.public_ip_address
						bastion_port = "22"
						bastion_user = "opc"
						bastion_private_key = file(var.private_key_oci)
				}
		inline = ["sudo /bin/su -c \"chown root /home/opc/iscsiattach.sh\"",
					"sudo /bin/su -c \"chmod u+x /home/opc/iscsiattach.sh\"",
					"sudo /bin/su -c \"/home/opc/iscsiattach.sh\""]
		}
		
		}
		
		
		resource "null_resource" "ExatedWebserver1_oci_u01_fstab" {
		depends_on = [null_resource.ExatedWebserver1_oci_iscsi_attach]
		
		provisioner "remote-exec" {
				connection {
						type     = "ssh"
						user     = "opc"
						host     = data.oci_core_vnic.ExatedWebserver1_VNIC1.private_ip_address
						private_key = file(var.private_key_oci)
						script_path = "/home/opc/myssh.sh"
						agent = false
						timeout = "10m"
						bastion_host = data.oci_core_vnic.ExatedBastionServer_VNIC1.public_ip_address
						bastion_port = "22"
						bastion_user = "opc"
						bastion_private_key = file(var.private_key_oci)
				}
		inline = ["sudo -u root parted /dev/sdb --script -- mklabel gpt",
					"sudo -u root parted /dev/sdb --script -- mkpart primary ext4 0% 100%",
					"sudo -u root mkfs.ext4 /dev/sdb1",
					"sudo -u root mkdir /u01",
					"sudo -u root mount /dev/sdb1 /u01",
					"sudo /bin/su -c \"echo '/dev/sdb1              /u01  ext4    defaults,noatime,_netdev    0   0' >> /etc/fstab\""
				]
		}
		
		}
		
###### File content - bastionserver.tf
		resource "oci_core_instance" "ExatedBastionServer" {
		availability_domain = lookup(data.oci_identity_availability_domains.ADs.availability_domains[0], "name")
		compartment_id = oci_identity_compartment.ExatedCompartment.id
		display_name = "ExatedBastionServer"
		shape = var.Shapes[0]
		subnet_id = oci_core_subnet.ExatedBastionSubnet.id
		source_details {
			source_type = "image"
			source_id   = lookup(data.oci_core_images.OSImageLocal.images[0], "id")
		}
		metadata = {
			ssh_authorized_keys = file(var.public_key_oci)
		}
		create_vnic_details {
			subnet_id = oci_core_subnet.ExatedBastionSubnet.id
			assign_public_ip = true
			nsg_ids = [oci_core_network_security_group.ExatedSSHSecurityGroup.id]
		}
		}
		
		data "oci_core_vnic_attachments" "ExatedBastionServer_VNIC1_attach" {
		availability_domain = lookup(data.oci_identity_availability_domains.ADs.availability_domains[0], "name")
		compartment_id = oci_identity_compartment.ExatedCompartment.id
		instance_id = oci_core_instance.ExatedBastionServer.id
		}
		
		data "oci_core_vnic" "ExatedBastionServer_VNIC1" {
		vnic_id = data.oci_core_vnic_attachments.ExatedBastionServer_VNIC1_attach.vnic_attachments.0.vnic_id
		}
		
		output "ExatedBastionServer_PublicIP" {
		value = [data.oci_core_vnic.ExatedBastionServer_VNIC1.public_ip_address]
		}
		
###### File content - dbsystem_UPDATED.tf
		resource "oci_database_db_system" "ExatedDBSystem" {
		availability_domain = lookup(data.oci_identity_availability_domains.ADs.availability_domains[0], "name")
		compartment_id = oci_identity_compartment.ExatedCompartment.id
		cpu_core_count = var.CPUCoreCount
		database_edition = var.DBEdition
		db_home {
			database {
			admin_password = var.DBAdminPassword
			db_name = var.DBName
			character_set = var.CharacterSet
			ncharacter_set = var.NCharacterSet
			db_workload = var.DBWorkload
			pdb_name = var.PDBName
			}
			db_version = var.DBVersion
			display_name = var.DBHomeDisplayName
		}
		disk_redundancy = var.DBDiskRedundancy
		shape = var.DBNodeShape
		subnet_id = oci_core_subnet.ExatedDBSubnet.id
		ssh_public_keys = [file(var.public_key_oci)]
		display_name = var.DBSystemDisplayName
		domain = var.DBNodeDomainName
		hostname = var.DBNodeHostName
		nsg_ids = [oci_core_network_security_group.ExatedDBSystemSecurityGroup.id]
		data_storage_percentage = "40"
		data_storage_size_in_gb = var.DataStorageSizeInGB
		license_model = var.LicenseModel
		node_count = var.NodeCount
		}
		
		data "oci_database_db_homes" "primarydb_home" {
		compartment_id = oci_identity_compartment.ExatedCompartment.id
		db_system_id   = "${oci_database_db_system.ExatedDBSystem.id}"
		
		filter {
			name   = "display_name"
			values = [var.DBHomeDisplayName]
		}
		}
		
		data "oci_database_databases" "primarydb" {
		compartment_id = oci_identity_compartment.ExatedCompartment.id
		db_home_id     = "${data.oci_database_db_homes.primarydb_home.db_homes.0.db_home_id}"
		}
		
		resource "oci_database_data_guard_association" "ExatedDBSystemStandby" {
			creation_type = "NewDbSystem"
			database_admin_password = var.DBAdminPassword
			database_id = data.oci_database_databases.primarydb.databases.0.id
			protection_mode = "MAXIMUM_PERFORMANCE"
			transport_type = "ASYNC"
			delete_standby_db_home_on_delete = "true"
		
			availability_domain = lookup(data.oci_identity_availability_domains.ADs.availability_domains[0], "name")
			display_name = var.DBStandbySystemDisplayName
			hostname = var.DBStandbyNodeHostName
			nsg_ids = [oci_core_network_security_group.ExatedDBSystemSecurityGroup.id]
			shape = var.DBStandbyNodeShape
			subnet_id = oci_core_subnet.ExatedDBSubnet.id
		}
		
		
		data "oci_database_db_nodes" "DBNodeList" {
		compartment_id = oci_identity_compartment.ExatedCompartment.id
		db_system_id = oci_database_db_system.ExatedDBSystem.id
		}
		
		data "oci_database_db_node" "DBNodeDetails" {
		db_node_id = lookup(data.oci_database_db_nodes.DBNodeList.db_nodes[0], "id")
		}
		
		data "oci_core_vnic" "ExatedDBSystem_VNIC1" {
		vnic_id = data.oci_database_db_node.DBNodeDetails.vnic_id
		}
		
		output "ExatedDBServer_PrivateIP" {
		value = [data.oci_core_vnic.ExatedDBSystem_VNIC1.private_ip_address]
		}
		
		#TO EXECUTE TERRRAFORM YOU NEED ANOTHER FILE ASKED setup_oci_tf_var.sh
		#CHECK THE DOCUMENT https://github.com/danilo19ee/zdatabase/blob/master/TERRAFORM/TERRAFORM%20SETUP%20ORACLE%20CLOUD%20OCI%20ENVIRONMENT
		root@raspberrypi:~/tf_oci/LESSON6_local_block_volumes-Exated# cp /root/setup_oci_tf_var.sh .
		root@raspberrypi:~/tf_oci/LESSON6_local_block_volumes-Exated# source setup_oci_tf_var.sh
		
### RUN TERRAFORM

###### INIT

		root@raspberrypi:~/tf_oci/dbsystem_with_dataguard-Exated# terraform init
		
		Initializing the backend...
		
		Initializing provider plugins...
		- Finding latest version of hashicorp/null...
		- Finding hashicorp/oci versions matching ">= 4.19.0"...
		- Installing hashicorp/null v3.1.0...
		- Installed hashicorp/null v3.1.0 (signed by HashiCorp)
		- Installing hashicorp/oci v4.21.0...
		- Installed hashicorp/oci v4.21.0 (signed by HashiCorp)
		
		Terraform has created a lock file .terraform.lock.hcl to record the provider
		selections it made above. Include this file in your version control repository
		so that Terraform can guarantee to make the same selections by default when
		you run "terraform init" in the future.
		
		
		Warning: Version constraints inside provider configuration blocks are deprecated
		
		on provider.tf line 2, in provider "oci":
		2:   version          = ">= 4.19.0" 
		
		Terraform 0.13 and earlier allowed provider version constraints inside the
		provider configuration block, but that is now deprecated and will be removed
		in a future version of Terraform. To silence this warning, move the provider
		version constraint into the required_providers block.
		
		Terraform has been successfully initialized!
		
		You may now begin working with Terraform. Try running "terraform plan" to see
		any changes that are required for your infrastructure. All Terraform commands
		should now work.
		
		If you ever set or change modules or backend configuration for Terraform,
		rerun this command to reinitialize your working directory. If you forget, other
		commands will detect it and remind you to do so if necessary.
		
###### PLAN
		
		root@raspberrypi:~/tf_oci/dbsystem_with_dataguard-Exated# terraform plan
		
		An execution plan has been generated and is shown below.
		Resource actions are indicated with the following symbols:
		+ create
		<= read (data resources)

		------------------------------------------------------------------------
		
		Note: You didn't specify an "-out" parameter to save this plan, so Terraform
		can't guarantee that exactly these actions will be performed if
		"terraform apply" is subsequently run.
		
###### APPLY
		
		root@raspberrypi:~/tf_oci/dbsystem_with_dataguard-Exated# terraform apply
		
		
		An execution plan has been generated and is shown below.
		Resource actions are indicated with the following symbols:
		+ create
		<= read (data resources)
		
		Do you want to perform these actions?
		Terraform will perform the actions described above.
		Only 'yes' will be accepted to approve.
		
		Enter a value: 
		
		Apply complete! Resources: 55 added, 0 changed, 0 destroyed.
		
		Outputs:
		
		ExatedBastionServer_PublicIP = [
		"129.159.56.155",
		]
		ExatedDBServer_PrivateIP = [
		"10.0.4.28",
		]
		ExatedPublicLoadBalancer_Public_IP = [
		tolist([
			"132.226.240.200",
		]),
		]
		ExatedWebserver1_PrivateIP = [
		"10.0.1.39",
		]
		ExatedWebserver2_PrivateIP = [
		"10.0.1.252",
		]
		
###### DESTROY
		
		root@raspberrypi:~/tf_oci/dbsystem_with_dataguard-Exated# terraform destroy 
		
	
		An execution plan has been generated and is shown below.
		Resource actions are indicated with the following symbols:
		- destroy
		
		Destroy complete! Resources: 55 destroyed.
		
