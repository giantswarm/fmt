# Naming Resources

## IAM Resources

AWS operator needs a set of roles in customer accounts to be able to operator against the AWS API. Most of the customers need bounded policy to be security complaint. Here we define a convention to help them to achieve that and a same time have a coherent way to name the roles.

From now own all clusters roles we need to create should have this convention

- Common prefix `gs-`
- Follow with `cluster id`
- Ending with `purpose`

Example:

OLD worker role: `5md7e-worker-EC2-K8S-Role`
NEW worker role: `gs-5md7e-worker-EC2-K8S-Role`

That way in the policy given to the customer to prepare the account we can set in resources `gs-*`.