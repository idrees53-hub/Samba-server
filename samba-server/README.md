##In this file you will find how you can share the directory with samba server and received by samba client in linux OS.
## for your reference I have added one demo samba configuration file and demo fstab file 

#Pre-requisities

1) create one AWS account.
2) create two instances in same availability zone.
3) In this project I have used RedHat OS instance (rhel).
4) You can use any distribution there is no dependencies. 



#Steps to implement the samba server 

## Take login with your samba server instance and make sure you allow '445'  port in the security group 

#Run these commands on your samba server instance 

yum install samba -y
systemctl enable --now smb nmb 

## Now we have to create directory to share with the samba server 

mkdir /smbshare
semanage fcontext -a -t samba_share_t /smbshare
restorecon -vvRF /smbshare 

## Now create one user by which you will authenticate and authorize your shared directory 

smbpasswd -a redhat ##I have given password redhat@123 here 

## Now write the configuration in /etc/samba/smb.conf file

[global]
        workgroup = IDREES
        security = user

[homes]
        comment = Home Directories
        valid users = %S, %D%w%S
        browseable = No
        read only = No
        inherit acls = Yes

[printers]
        comment = All Printers
        path = /var/tmp
        printable = Yes
        create mask = 0600
        browseable = No

[print$]
        comment = Printer Drivers
        path = /var/lib/samba/drivers
        write list = @printadmin root
        force group = @printadmin
        create mask = 0664
        directory mask = 0775

[smbshare]
        comment = Shared Directory
        path = /smbshare
        browseable = yes
        read only = no
        writeable = yes
        valid users = redhat

## you can copy the whole configuration it will work fine 
## now restart the service again 

systemctl restart smb nmb 


## Now just simply take remote access with your samba client instance

## install this necessary package

yum install cifs-utils -y 

## Now create directory on which you want to mount the shared directory 

mkdir /smbmount 

## Now just simply open your /etc/fstab file and make this entry  
## and let's assume that samba server ip is 198.168.0.1 now..

//198.168.0.1/smbshare	/smbmount 	cifs 	username=redhat,password=redhat@123 0 0 

## now save this file and run mount -a command 

mount -a 

## now checkout your samba shared directory 

df -h 


## congratulations your samba server is successfully configured 





- 
