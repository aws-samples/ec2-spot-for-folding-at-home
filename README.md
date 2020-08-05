## Researching treatments for infectious disease with Folding@home and Amazon EC2 Spot Instances
This GitHub contains an AWS CloudFormation template along with step-by-step instructions for deploying EC2 Spot Instances - optimized for the latest Folding@home client software. We’ll leverage EC2 Spot best practices to be flexible across a combination of GPU-optimized Amazon EC2 Spot Instances configured in an Amazon EC2 Auto Scaling group (ASG) to handle launching and maintaining a desired capacity, and to automatically request resources to replace any that are interrupted or manually terminated. 

**What you’ll build**
* An Amazon Virtual Private Cloud (VPC) configured with public and private subnets according to AWS best practices. 
* Identity and Access Management (IAM) roles to manage permissions for Amazon EC2 Auto Scaling.
* Security Group for the EC2 Spot instances to control inbound and outbound traffic
* Auto Scaling Group for scaling EC2 Spot instances in and out as needed using the Capacity Optimized allocation strategy.
* Amazon CloudWatch instance metrics and logs for real-time monitoring of our protein folding progress.
 
 ![FAH](https://user-images.githubusercontent.com/68295015/89449052-5fcc6980-d70d-11ea-9419-e14cafad48e0.png)
 
*To complete the setup, you must have an Amazon Web Services (AWS) account with permissions to the listed resources above. When you sign up for AWS, your AWS account is automatically signed up for all services in AWS, including Amazon EC2. If you don't have an AWS account, find more info about creating an account here.*

### Costs and Licensing
The AWS CloudFormation CFn template includes customizable configuration parameters. Some of these settings, such as instance type, will affect the cost of deployment. For cost estimates, see the pricing pages for each AWS service you will be using. Prices are subject to change. You are responsible for the cost of the AWS services used. There is no additional cost for using the CFn template.

**Note:** _There is no additional charge to use Deep Learning AMIs — you pay only for the AWS resources while they’re running. Folding@home client software is free, open-source software that is distributed under the Folding@home EULA._

## How to Deploy 
### Part One: Download and configure the CFn template.

First thing we’ll need to do is download, then make a few edits to the template. 
Once downloaded, open the template file in your favorite text editor so we can make a few edits to the configuration before deploying.

In the User Information section, we have the option to create a unique username, join or create a new team, or contribute anonymously. For this example, I’ll leave the values set to default and contribute as an anonymous user, the default team. More details about [teams and leaderboards](https://foldingathome.org/support/faq/stats-teams-usernames/)  can be found here and details about [PASSKEYs here](https://foldingathome.org/support/faq/points/passkey/). 

```bash
<!-- User Information --> 
<team v="Default"/>
<user v="0"/>
<passkey v="PASSKEY_HERE"/>
```
 
Once edited and saved to a location we can easily find later, let’s move on to the next section where we’ll upload the template in the AWS CloudFormation console. 

### Part Two: Launching the Stack
Next, we’ll log into the AWS console, Select the region you want to run the solution in, then navigate to AWS CloudFormation to launch our template.

In the CloudFormation console, click on create stack. Upload the template we just configured and click on Next to specify stack details.

<img width="1680" alt="create-stack" src="https://user-images.githubusercontent.com/68295015/89452622-d15ae680-d712-11ea-8c47-b97a7b55a21d.png"> 

Enter a stack name and adjust the capacity parameters as needed. In this example I set the desiredCapacity and minSize at 2 to handle protein folding jobs assigned to our client, and then the maxSize set at 12 in case we need to scale capacity for larger jobs that get assigned. These parameters can be adjusted based on your desired capacity. 


 ![Specify Stack Details](https://user-images.githubusercontent.com/68295015/89452835-25fe6180-d713-11ea-922c-c325a6b15367.png)
 

In the next stack configuration step, you can optionally add additional configurations like; tags, permissions, stack policies, rollback options and more advanced options. Adding tags will make it easier for us to break out usage and cost data later if needed. Click Next to Review and then create the stack. 

Under the events tab, you can see the status of the AWS resources being created. When the status is CREATE_COMPLETE (approx. 3-5 minutes), the environment with Folding@home installed is ready. Once the stack is created, the GPU instances will begin protein simulation.

![cluster_details](https://user-images.githubusercontent.com/68295015/89453215-c2286880-d713-11ea-85f0-8aefb08bdfd3.png)


#### Results
The Amazon CloudFormation template creates a log group “fahlog” that each of our instances will send log data to, that will allow us to visualize the protein folding progress in near real-time via the Amazon CloudWatch console. To see the log data, navigate over to the “Resources” tab and click on the cloudWatchLogGroup link for ‘fahlog’. Alternatively, you can navigate to the Amazon CloudWatch Console and select ‘fahlog’ under log groups. Note: Sometimes it takes a bit of time for Folding@Home Work Units (WU) to be downloaded in the instances and allocate all the available GPUs. 


![logs](https://user-images.githubusercontent.com/68295015/89453519-40850a80-d714-11ea-9099-5f8d7e1e266d.png)

In the CloudWatch console, let’s check out the ‘Insights’ feature in the left navigation menu to see analytics for our protein folding logs. Select fahlog in the search field and run the default query that is provided for you in the query editor window to show our protein folding results.

![log results](https://user-images.githubusercontent.com/68295015/89453754-9194fe80-d714-11ea-8254-fd8eb0a407b7.png)
 
Another thing we can do is create a [Dashboard](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Dashboards.html) in the CloudWatch console that will automatically refresh based on the time intervals we set. Under Dashboards in the left Navigation bar, I was able to quickly whip up a few widgets to visualize CPU utilization, Network in/out, and protein folding completed steps. This is a pretty nifty tool that with a little more time, you could configure more detailed metrics like cost per fold, and GPU monitoring. 


![cw_dashboard](https://user-images.githubusercontent.com/68295015/89454290-57782c80-d715-11ea-9afb-0837d78ee9d2.png)

### Part three: Clean up**
You can let this run as long as you want to contribute to this project. When you’re ready to stop, CloudFormation gives us the option to delete the stack and resources created. On the CloudFormation console, select the stack, and select delete. When you delete a stack, you delete the stack and all of its resources.

## Conclusion
In this post, we launched a cluster of EC2 GPU-optimized Spot instances to aid in Folding@home’s protein dynamics research that could lead to therapeutics for infectious diseases. We leveraged Spot best practices by being flexible with instance selections across multiple families, sizes, and Availability Zones and by selecting the capacity-optimized allocation strategy to ensure our cluster scales optimally and securely. 

## About the project
> Folding@home (FAH) is a distributed computing project that uses computational modeling to simulate protein structure, stability, and shape (how it folds). These simulations help to advance drug discoveries and cures for diseases linked to protein dynamics within human cells. The FAH software crowdsources its distributed compute platform allowing anyone to contribute by donating unused computational resources from personal computers, laptops, and cloud servers. 

> [EC2 Spot Instances](https://aws.amazon.com/ec2/spot/) are spare EC2 capacity available at up to a 90% discount compared to On-Demand prices. The only difference between On-Demand and Spot Instances is that Spot Instances can be interrupted by EC2 with two minutes of notification when EC2 needs the capacity back. This makes Spot Instances a great fit for stateless, fault-tolerant workloads like big data, containers, batch processing, AI/ML training, CI/CD and test/dev. For more information, see Amazon EC2 Spot Instances. 



## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.

