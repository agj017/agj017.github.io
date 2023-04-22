1. ssh -i your_key.pem ec2-user@instance_public_dns 
    ex) sudo ssh -i ./onion-dev.pem ec2-user@ec2-18-143-155-131.ap-southeast-1.compute.amazonaws.com

2. psql -h onion-dev-postgres.cl3kdqdm7erq.ap-southeast-1.rds.amazonaws.com -d onion -U postgres
    

