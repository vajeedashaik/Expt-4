# Experiment  Create-a-datacenter-with-two-hosts-and-run-two-cloudlets-on-it.
Create a datacenter with two hosts and run two cloudlets on it. The cloudlets run in VMswith Different MIPS requirements. The cloudlets will take different time to complete the execution Depending on the requested VM performance.
 
## Overview
This project demonstrates a CloudSim 3.0.3 simulation experiment where two cloudlets are executed on virtual machines with different MIPS values.

Since the virtual machines have different computational capacities, the cloudlets complete execution at different times depending on VM performance.

This experiment is commonly used in cloud computing laboratory exercises, academic coursework, and performance analysis studies.

---

## Aim
To create a datacenter with two hosts and execute two cloudlets on virtual machines having different MIPS requirements such that cloudlets take different execution times.

---

## Objectives
- To simulate a cloud datacenter using CloudSim
- To create multiple hosts within a datacenter
- To configure virtual machines with different MIPS values
- To execute cloudlets with identical workloads
- To analyze execution time differences

---

## Tools and Technologies

| Tool | Version |
|------|---------|
| Java JDK | 1.8 |
| CloudSim | 3.0.3 |
| IDE | IntelliJ IDEA / Eclipse |
| Operating System | Windows / Linux |

---

## System Configuration

### Datacenter Configuration

| Parameter | Value |
|------------|-------|
| Datacenters | 1 |
| Hosts | 2 |
| RAM | 4096 MB |
| Storage | 1,000,000 MB |
| Bandwidth | 10,000 Mbps |
| VM Scheduler | Time Shared |

---

### Virtual Machine Configuration

| Parameter | VM1 | VM2 |
|------------|------|------|
| MIPS | 500 | 2000 |
| RAM | 512 MB | 512 MB |
| Bandwidth | 1000 Mbps | 1000 Mbps |
| VMM | Xen | Xen |
| Scheduler | Time Shared | Time Shared |

---

### Cloudlet Configuration

| Parameter | Value |
|------------|--------|
| Cloudlets | 2 |
| Length | 40,000 MI |
| File Size | 300 MB |
| Output Size | 300 MB |
| Processing Elements | 1 |

---

## Program File

```java
/*
 * Title:        CloudSim Toolkit
 * Description:  CloudSim (Cloud Simulation) Toolkit for Modeling and Simulation
 *               of Clouds
 * Licence:      GPL - http://www.gnu.org/copyleft/gpl.html
 *
 * Copyright (c) 2009, The University of Melbourne, Australia
 */

package org.cloudbus.cloudsim.examples;

import org.cloudbus.cloudsim.*;
import org.cloudbus.cloudsim.core.CloudSim;
import org.cloudbus.cloudsim.provisioners.BwProvisionerSimple;
import org.cloudbus.cloudsim.provisioners.PeProvisionerSimple;
import org.cloudbus.cloudsim.provisioners.RamProvisionerSimple;

import java.text.DecimalFormat;
import java.util.ArrayList;
import java.util.Calendar;
import java.util.LinkedList;
import java.util.List;


/**
 * A simple example showing how to create
 * two datacenters with one host each and
 * run two cloudlets on them.
 */
public class CloudSimExample4 {
	public static DatacenterBroker broker;

	/** The cloudlet list. */
	private static List<Cloudlet> cloudletList;

	/** The vmlist. */
	private static List<Vm> vmlist;

	/**
	 * Creates main() to run this example
	 */
	public static void main(String[] args) {

		Log.println("Starting CloudSimExample4...");

		try {
			// First step: Initialize the CloudSim package. It should be called
			// before creating any entities.
			int num_user = 1;   // number of cloud users
			Calendar calendar = Calendar.getInstance();
			boolean trace_flag = false;  // mean trace events

			// Initialize the GridSim library
			CloudSim.init(num_user, calendar, trace_flag);

			// Second step: Create Datacenters
			//Datacenters are the resource providers in CloudSim. We need at list one of them to run a CloudSim simulation
			Datacenter datacenter0 = createDatacenter("Datacenter_0");
			Datacenter datacenter1 = createDatacenter("Datacenter_1");

			//Third step: Create Broker
			broker = createBroker();
			int brokerId = broker.getId();

			//Fourth step: Create one virtual machine
			vmlist = new ArrayList<>();

			//VM description
			int mips = 250;
			long size = 10000; //image size (MB)
			int ram = 512; //vm memory (MB)
			long bw = 1000;
			int pesNumber = 1; //number of cpus
			String vmm = "Xen"; //VMM name

			//create two VMs
			Vm vm1 = new Vm(brokerId, mips, pesNumber, ram, bw, size, vmm, new CloudletSchedulerTimeShared());
			Vm vm2 = new Vm(brokerId, mips, pesNumber, ram, bw, size, vmm, new CloudletSchedulerTimeShared());

			//add the VMs to the vmList
			vmlist.add(vm1);
			vmlist.add(vm2);

			//submit vm list to the broker
			broker.submitGuestList(vmlist);


			//Fifth step: Create two Cloudlets
			cloudletList = new ArrayList<>();

			//Cloudlet properties
			long length = 40000;
			long fileSize = 300;
			long outputSize = 300;
			UtilizationModel utilizationModel = new UtilizationModelFull();

			Cloudlet cloudlet1 = new Cloudlet(length, pesNumber, fileSize, outputSize, utilizationModel, utilizationModel, utilizationModel);
			cloudlet1.setUserId(brokerId);

			Cloudlet cloudlet2 = new Cloudlet(length, pesNumber, fileSize, outputSize, utilizationModel, utilizationModel, utilizationModel);
			cloudlet2.setUserId(brokerId);

			//add the cloudlets to the list
			cloudletList.add(cloudlet1);
			cloudletList.add(cloudlet2);

			//submit cloudlet list to the broker
			broker.submitCloudletList(cloudletList);


			//bind the cloudlets to the vms. This way, the broker
			// will submit the bound cloudlets only to the specific VM
			broker.bindCloudletToVm(cloudlet1.getCloudletId(),vm1.getId());
			broker.bindCloudletToVm(cloudlet2.getCloudletId(),vm2.getId());

			// Sixth step: Starts the simulation
			CloudSim.startSimulation();


			// Final step: Print results when simulation is over
			List<Cloudlet> newList = broker.getCloudletReceivedList();

			CloudSim.stopSimulation();

        	printCloudletList(newList);

			Log.println("CloudSimExample4 finished!");
		}
		catch (Exception e) {
			e.printStackTrace();
			Log.println("The simulation has been terminated due to an unexpected error");
		}
	}

	private static Datacenter createDatacenter(String name){

		// Here are the steps needed to create a PowerDatacenter:
		// 1. We need to create a list to store
		//    our machine
		List<Host> hostList = new ArrayList<>();

		// 2. A Machine contains one or more PEs or CPUs/Cores.
		// In this example, it will have only one core.
		List<Pe> peList = new ArrayList<>();

		int mips = 1000;

		// 3. Create PEs and add these into a list.
		peList.add(new Pe(new PeProvisionerSimple(mips))); // need to store Pe id and MIPS Rating

		//4. Create Host with its id and list of PEs and add them to the list of machines
		int ram = 2048; //host memory (MB)
		long storage = 1000000; //host storage
		int bw = 10000;

		//in this example, the VMAllocatonPolicy in use is SpaceShared. It means that only one VM
		//is allowed to run on each Pe. As each Host has only one Pe, only one VM can run on each Host.
		hostList.add(
    			new Host(
    				new RamProvisionerSimple(ram),
    				new BwProvisionerSimple(bw),
    				storage,
    				peList,
    				new VmSchedulerSpaceShared(peList)
    			)
    		); // This is our first machine

		// 5. Create a DatacenterCharacteristics object that stores the
		//    properties of a data center: architecture, OS, list of
		//    Machines, allocation policy: time- or space-shared, time zone
		//    and its price (G$/Pe time unit).
		String arch = "x86";      // system architecture
		String os = "Linux";          // operating system
		String vmm = "Xen";
		double time_zone = 10.0;         // time zone this resource located
		double cost = 3.0;              // the cost of using processing in this resource
		double costPerMem = 0.05;		// the cost of using memory in this resource
		double costPerStorage = 0.001;	// the cost of using storage in this resource
		double costPerBw = 0.0;			// the cost of using bw in this resource
		LinkedList<Storage> storageList = new LinkedList<>();	//we are not adding SAN devices by now

	       DatacenterCharacteristics characteristics = new DatacenterCharacteristics(
	                arch, os, vmm, hostList, time_zone, cost, costPerMem, costPerStorage, costPerBw);


		// 6. Finally, we need to create a PowerDatacenter object.
		Datacenter datacenter = null;
		try {
			datacenter = new Datacenter(name, characteristics, new VmAllocationPolicySimple(hostList), storageList, 0);
		} catch (Exception e) {
			e.printStackTrace();
		}

		return datacenter;
	}

	//We strongly encourage users to develop their own broker policies, to submit vms and cloudlets according
	//to the specific rules of the simulated scenario
	private static DatacenterBroker createBroker(){

		DatacenterBroker broker = null;
		try {
			broker = new DatacenterBroker("Broker");
		} catch (Exception e) {
			e.printStackTrace();
			return null;
		}
		return broker;
	}

	/**
	 * Prints the Cloudlet objects
	 * @param list  list of Cloudlets
	 */
	private static void printCloudletList(List<Cloudlet> list) {
		int size = list.size();
		Cloudlet cloudlet;

		String indent = "    ";
		Log.println();
		Log.println("========== OUTPUT ==========");
		Log.println("Cloudlet ID" + indent + "STATUS" + indent +
				"Data center ID" + indent + "VM ID" + indent + "Time" + indent + "Start Time" + indent + "Finish Time");

		DecimalFormat dft = new DecimalFormat("###.##");
		for (Cloudlet value : list) {
			cloudlet = value;
			Log.print(indent + cloudlet.getCloudletId() + indent + indent);

			if (cloudlet.getStatus() == Cloudlet.CloudletStatus.SUCCESS) {
				Log.print("SUCCESS");

				Log.println(indent + indent + cloudlet.getResourceId() + indent + indent + indent + cloudlet.getGuestId() +
						indent + indent + dft.format(cloudlet.getActualCPUTime()) + indent + indent + dft.format(cloudlet.getExecStartTime()) +
						indent + indent + dft.format(cloudlet.getExecFinishTime()));
			}
		}

	}
}

```

## Algorithm

1. Initialize the CloudSim library.
2. Create a datacenter with two hosts.
3. Configure host resources.
4. Create a datacenter broker.
5. Create two virtual machines with different MIPS values.
6. Submit the VM list to the broker.
7. Create two cloudlets with identical workloads.
8. Assign cloudlets to VMs.
9. Start the CloudSim simulation.
10. Collect execution results and stop the simulation.

---

## Execution Time Formula

Execution Time = Cloudlet Length / VM MIPS

---

## Sample Output

<img width="716" height="373" alt="image" src="https://github.com/user-attachments/assets/2784d029-c7d6-4a35-bb47-a1871cb5acf8" />


## Result

The experiment successfully demonstrates that cloudlets executed on higher MIPS virtual machines complete faster than those executed on lower MIPS VMs, even when the workload is identical.

---

## Conclusion

This simulation validates that CloudSim scheduling accurately reflects performance differences in heterogeneous cloud environments. Higher MIPS virtual machines reduce execution time for identical workloads.




