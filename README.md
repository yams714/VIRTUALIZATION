
provider "azurerm" {
  features = {}
}

resource "azurerm_resource_group" "rg" {
  name     = var.resource_group
  location = var.location
}

resource "azurerm_virtual_network" "vnet" {
  name                = var.virtual_network_name
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
}

resource "azurerm_subnet" "subnet" {
  name                 = var.subnet_name
  resource_group_name  = azurerm_resource_group.rg.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = ["10.0.1.0/24"]
}

resource "azurerm_network_interface" "nic" {
  name                = var.network_interface_name
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name

  ip_configuration {
    name                          = "nicConfig"
    subnet_id                     = azurerm_subnet.subnet.id
    private_ip_address_allocation = "Dynamic"
  }
}

resource "azurerm_windows_virtual_machine" "vm" {
  name                  = var.virtual_machine_name
  resource_group_name   = azurerm_resource_group.rg.name
  location              = azurerm_resource_group.rg.location
  size                  = var.virtual_machine_size
  admin_username        = var.admin_username
  admin_password        = var.admin_password
  network_interface_ids = [azurerm_network_interface.nic.id]

  source_image_reference {
    publisher = "MicrosoftWindowsServer"
    offer     = "WindowsServer"
    sku       = "2019-Datacenter"
    version   = "latest"
  }
}

variable "resource_group" {
  type    = string
  default = "myResourceGroup"
}

variable "location" {
  type    = string
  default = "East US"
}

variable "virtual_network_name" {
  type    = string
  default = "myVirtualNetwork"
}

variable "subnet_name" {
  type    = string
  default = "mySubnet"
}

variable "network_interface_name" {
  type    = string
  default = "myNetworkInterface"
}

variable "virtual_machine_name" {
  type    = string
  default = "myWindowsVM"
}

variable "virtual_machine_size" {
  type    = string
  default = "Standard_DS1_v2"
}

variable "admin_username" {
  type    = string
  default = "adminuser"
}

variable "admin_password" {
  type    = string
  default = "P@ssw0rd123!"
}
