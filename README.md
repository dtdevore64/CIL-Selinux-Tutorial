# **CIL Selinux Tutorial**
<br><br>

I will show you how to write Selinux policy using the CIL(Common Intermediate Language). We will write the policy to confine a simple program. Our program will be the ```xcowsay``` program.
<br><br>




***Step 1.*** Before we start writing policy you need to make sure to put Selinux into permissive mode so we can test the policy without it getting blocked every time because it was in enforcing mode. To put it into permissive mode just run the following:

```
sudo setenforce 0
```

<br><br><br>



***Step 2.*** Create the file called ```xcowsay.cil``` 

```
touch xcowsay.cil
nano xcowsay.cil
```
<br><br><br>


***Step 3.*** Create the file context for our file

```
(filecon "/usr/bin/xcowsay" file (staff_u object_r xcowsay_exec_t ((s0) (s0))))
```

<br><br><br>


***Step 4.*** Declare our process type and executable type

```
(type xcowsay_t)
(type xcowsay_exec_t)
```

<br><br><br>


***Step 5.*** Associate our process type and executable type with the ```object_r``` role type

```
(roletype object_r xcowsay_t)
(roletype object_r xcowsay_exec_t)

```
<br><br><br>


***Step 6.*** Associate our current role on the system as a logged in user which in this case is ```staff_r``` to the process type

``` 
(roletype staff_r xcowsay_t)
    
```
<br><br><br>


***Step 7.*** Add rules that are equivalent to the ```application_domain``` interface in Refpolicy

```
(typeattributeset application_domain_type (xcowsay_t))
(typeattributeset application_exec_type (xcowsay_exec_t))
(typeattributeset domain (xcowsay_t))
(typeattributeset entry_type (xcowsay_exec_t))

```
<br><br><br>


***Step 8.*** Add allow rules so that we can do a typetransition

```
(allow xcowsay_t xcowsay_exec_t (file (entrypoint ioctl read getattr lock map execute open)))
(allow staff_t xcowsay_exec_t (file (ioctl read getattr map execute open execute_no_trans)))
(allow staff_t xcowsay_t (process (transition)))
(typetransition staff_t xcowsay_exec_t process xcowsay_t)
```

<br><br><br>


***Step 9.*** Change the context of the ```xcowsay``` program to match what we have in our ```.cil``` file and make sure it is labeled correct

```
sudo chcon -t xcowsay_exec_t /usr/bin/xcowsay
ls -lZ /usr/bin/xcowsay
```
<br><br><br>


***Step 10.*** Build and Install our module like so:

```
sudo semodule -i xcowsay.cil
```

<br><br><br>


***Step 10.*** Run the ```xcowsay``` program and check the logs to see what AVC denials we get and decide on whether to add them or not to the policy. To do that run the following command

```
xcowsay
sudo ausearch -m AVC,USER_AVC,SELINUX_ERR,USER_SELINUX_ERR -ts 16:18
```

<br><br><br>




***Step 11.*** These are the allow rules I got from checking the AVC denials from the previous step and decided they were ok to add to the policy

```
(allow xcowsay_t staff_t (fd (use)))
(allow xcowsay_t staff_t (fifo_file (ioctl read write getattr lock append)))
(allow xcowsay_t staff_t (process (sigchld)))
(allow xcowsay_t staff_t (unix_stream_socket (connectto)))
(allow xcowsay_t fs_t (filesystem (getattr)))
```

<br><br><br><br>



***Step 12.*** Run the ```xcowsay``` program again and you should get no AVC denials. Put it back into enforcing mode and run the program again to make sure it doesn't get blocked. In our case it does not get blocked once set to enforcing so there is nothing more and our policy is done!

```
sudo setenforce 1
xcowsay
```




