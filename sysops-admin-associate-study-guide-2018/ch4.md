# Chapter 4 Compute
## Exercise 4.1 Create a Linux Instance via AWS Management Console
### Up
1. Launch Amazon EC2
1. Choose Linux AMI
1. T.2 micro instance type (free tier eligible)
1. Launch into default VPC
1. Assign a public IP address
1. Add tag to instance of `key` = `Yankee`, `value` = `Zulu`
1. Create new security group called `YankeeZuluSG`
1. Add rule to `YankeeZulu` security group that allows SSH access from anywhere (0.0.0.0/0)
1. Launch instance
1. Create new key pair, name it `YankeeZuluKeyPair`, save private portion
1. SSH into instance
  1. You may need to `chmod 400` the newly downloaded keypair
1. Install software updates
  1. Maybe you want to start a simple web server; assuming Ubuntu OS...
  1. Add HTTP inbound rule to `YankeeZuluSG` from anywhere
  1. `mkdir www && cd www`
  1. `touch index.html`
  1. 
```
cat <<HEREDOC > index.html
<html>
  <head><title>Test HTML page</title></head>
  <body><h3>Test HTML page</h3></body>
</html>
HEREDOC
``` 
  1. `sudo python3 -m http.server 80`  
1. Run `ifconfig -a` then make note of IP address
1. Access instance metadata by running `curl http://169.254.169.254/latest/meta-data/public-ipv4`
1. Exist SSH session

### Down
If not using for Exercise 4.6
1. Terminate instance
1. Delete security group called `YankeeZuluSG`
1. Delete keypair `YankeeZuluKeyPair`

