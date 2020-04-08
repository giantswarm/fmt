# Naming Resources

## IAM Resources

AWS operator needs a set of roles in customer accounts to be able to act 
against the AWS API. Most of the customers need a bounded policy to be 
security complaint. Here we define a convention to help them to achieve 
this and at the same time have a coherent way to name the roles.

From now on all clusters roles we need to create should have this convention

- Common prefix (`gs`).
- Cluster ID.
- Purpose in lower case.

Example:

- OLD worker role: `5md7e-master-EC2-K8S-Role`
- NEW worker role: `gs-5md7e-master-ec2-k8s` (prefix `gs`, cluster ID `5md7e`, 
purpose `master-ec2-k8s`) 

That way in the policy given to the customer to prepare the account we can set 
in resources `gs-*`.
