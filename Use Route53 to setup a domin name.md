Use Route53 to map the ingress Address and create a record of type A. In my case it would be mycluster.myprojectssatya.click

Prerequisities


1) Buy a domain name from Route53, look for .click they are the cheapest.

2) Create a hostedzone.
   
# Associate your domain name to ingress

 After you have created your Route53 domain, in the subdomain section type the name your cluster is to be associated and make sure to check the ALIAS button. It is used to map to an ALB.
