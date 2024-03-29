# WTESTBEDS 2023

Official repository for the paper "[FABRIC Testbed from the Eyes of a Network Researcher](https://doi.org/10.5753/wtestbeds.2023.230665)" to [WTESTBEDS 2023 - 2º Workshop de Testbeds](https://csbc.sbc.org.br/2023/wtestbeds/).

# Overview

This repository serves as a comprehensive archive of the data and associated command scripts presented in the article "FABRIC Testbed from the Eyes of a Network Researcher," originally submitted to the  WTESTBEDS 2023 - 2nd Testbeds Workshop.

# FABRIC

FABRIC testbed[^1] has been designed to provide a unique national research infrastructure to enable cutting-edge and exploratory research at-scale in networking.

## Slice definition: requirements and allocation

<p align="center">
    <img src="assets/topology.png" alt="Topology of FABRIC Testbed" width="800">
</p>
<p align="center">
    <em>Topology of FABRIC Testbed</em>
</p>

## Booking a slice

To book a slice in FABRIC Testbed and use its metaresearch functionalities, the experimenter needs to import the “FABlib library” (v. 1.4.3) that provides its API primitives. For example, one may query for available testbed resources and settings by using ```fablib.list_sites()``` and then get 3 random sites that are available for experimentation with the command ```fablib.get_random_sites(count=3)```. There is also a geographic location parameter to select sites located between Los Angeles and New York

```python
from fabrictestbed_extensions.fablib.fablib import FablibManager as fablib_manager

fablib = fablib_manager()
los_angelis_lat_long = (34.049043, -118.259476)
new_york_lat_long = (43.453157655429585, -76.53364473809769)

try:
    sites = fablib.get_random_sites(count=3, filter_function=lambda x:
                                        x['nic_connectx_5_available'] >= 2 and
                                        x['cores_available'] >= 2 and
                                        x['disk_available'] >= 10 and
                                        x['location'][1] > los_angelis_lat_long[1] and
                                        x['location'][1] < new_york_lat_long[1]
                                    )
    print(f"Available Sites: {sites}")
except Exception as e:
    print(f"Exception: {e}")
```

New components can be added to the slice. For example, a dedicated smartNIC (```command add_component()``` or a L2 link, see Figure "Slice visualization". After all the requirementsspecified for the experiment components , the request of the slice can be submitted.

```python
slice_name = 'OSPF_Routing_Topology'
model_name = 'NIC_Basic'
nic_name = ['nic1', 'nic2']
try:
    slice = fablib.new_slice(name=slice_name)
    
    nodes, nics = [], []
    
    for i, name in enumerate(sites):
        nodes.insert(i, slice.add_node(name=f'r{i+1}', site=name, image='default_debian_10'))
    
    for node in nodes:
        nics.insert(i, node.add_component(model=model_name, name=nic_name[0]).get_interfaces()[0])
        nics.insert(i+1, node.add_component(model=model_name, name=nic_name[1]).get_interfaces()[0])
    
    net1 = slice.add_l2network(name='net1', interfaces=[nics[0], nics[3]])
    net2 = slice.add_l2network(name='net2', interfaces=[nics[2], nics[5]])
    net3 = slice.add_l2network(name='net3', interfaces=[nics[4], nics[1]])
    
    slice.submit()
except Exception as e:
    print(f"Exception: {e}")
```

For the sake of simplicity, a tree-node ring will be here used to demonstrate network dynamic routing reacting to node failure, which was explored in the experiments.

<p align="center">
    <img src="assets/fabric_slice_visualization.png" alt="Slice visualization: ring topology connected with dedicated smartnics" width="800">
</p>
<p align="center">
    <em>Slice visualization: ring topology connected with dedicated smartnics</em>
</p>

## Components configuration and automation



## Transforming nodes in routers to support routing protocols

Free Range Routing[^2] (a.k.a. FRRouting or FRR), is a free and open source Internet routing protocol suite for Linux and Unix platforms. It implements BGP, OSPF, RIP, IS-IS, PIM, LDP, BFD, Babel, PBR, OpenFabric and VRRP, with alpha support for EIGRP and NHRP.

By default, the slice has a validity of 24 hours unless renewed. Below, it demonstrates how to delete a slice by name. It is an important practice to delete the slice after the experiments, in order to reduce the idle time of allocated resources.

```python
# Delete slice by name
try:
    slice_name = 'Triangle_Topology'
    slice = fablib.get_slice(name=slice_name)
    slice.delete()
except Exception as e:
    print(f"Exception: {e}")
```

# Running experiments in FABRIC testbed

The tests were conducted on the FABRIC platform using a Jupyter Notebook. Each node in the platform was equipped with 2 cores, 8 GB of RAM, and 10 GB of disk space. The operating system employed for the tests was Rocky Linux 8.5 (Green Obsidian), while the interfaces used were dedicated NICs “Mellanox ConnectX-5 Dual Port 10/25GbE”. iperf3[^3] (v. 3.5) was used to generate the traffic and measure the throughput.

## OSPF: failure recover use case

<p align="center">
    <img src="assets/failure_recovery.svg" alt="RTT with events of path failure and path recovery" width="800">
</p>
<p align="center">
    <em>RTT with events of path failure and path recovery</em>
</p>

## Throughput use case

<p align="center">
    <img src="assets/throughput_evaluation_default.svg" alt="Default OS/NIC config" width="800">
</p>
<p align="center">
    <em>Default OS/NIC config</em>
</p>

<p align="center">
    <img src="assets/throughput_evaluation_tuning.svg" alt="Tuning configuration" width="800">
</p>
<p align="center">
    <em>Tuning configuration</em>
</p>

# How to cite this work?

ABNT:

PONTES, Edgard da Cunha et al. FABRIC Testbed from the Eyes of a Network Researcher. In: WORKSHOP DE TESTBEDS, 2. , 2023, João Pessoa/PB. Anais [...]. Porto Alegre: Sociedade Brasileira de Computação, 2023 . p. 38-49. DOI: https://doi.org/10.5753/wtestbeds.2023.230665.

BibTeX:
```bibtex
@inproceedings{wtestbeds,
 author = {Edgard Pontes and Magnos Martinello and Cristina Dominicini and Marcos Schwarz and Moises Ribeiro and Everson Borges and Italo Brito and Jeronimo Bezerra and Marinho Barcellos},
 title = {FABRIC Testbed from the Eyes of a Network Researcher},
 booktitle = {Anais do II Workshop de Testbeds},
 location = {João Pessoa/PB},
 year = {2023},
 keywords = {},
 issn = {0000-0000},
 pages = {38--49},
 publisher = {SBC},
 address = {Porto Alegre, RS, Brasil},
 doi = {10.5753/wtestbeds.2023.230665},
 url = {https://sol.sbc.org.br/index.php/wtestbeds/article/view/24812}
}
```

RefWorks:
```refworks
@article{{wtestbeds}{},
	author = {Pontes, E., Martinello, M., Dominicini, C., Schwarz, M., Ribeiro, M., Borges, E., Brito, I., Bezerra, J., Barcellos, M.},
	title = {FABRIC Testbed from the Eyes of a Network Researcher},
	journal = {Anais do Workshop de Testbeds},

	year = {2023},
	doi = {10.5753/wtestbeds.2023.230665},
	url = {https://sol.sbc.org.br/index.php/wtestbeds/article/view/24812}
}
```

[^1]: BALDIN, Ilya et al. [Fabric: A national-scale programmable experimental network infrastructure](https://ieeexplore.ieee.org/abstract/document/8972790). IEEE Internet Computing, v. 23, n. 6, p. 38-47, 2019.
[^2]: [https://frrouting.org/](https://frrouting.org/)
[^3]: [https://iperf.fr/](https://iperf.fr/)
