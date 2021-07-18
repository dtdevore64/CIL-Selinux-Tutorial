# **Changing-Selinux-Refpolicy-to-CIL-Policy**
<br><br>

I will show you how to change your Selinux Refpolicy into CIL Policy with an example. The example program I have used is called xcowsay and I have written the policy in Refpolicy format and will translate it into CIL for you. You will see how much less policy there is and how easier it is to write.
<br><br>



***Step 1.*** Create your own policy module using the Refpolicy language which should have a ```".te"``` and a ```".fc"``` file. My example looks like the following

```
touch xcowsay.te
touch xcowsay.fc
```
<br><br><br>


***Step 2.*** We will start with writing the policy for the ```"te"``` file.

```
policy_module(xcowsay, 1.0.0)

type xcowsay_t;
type xcowsay_exec_t;
application_domain(xcowsay_t, xcowsay_exec_t)
role staff_r types xcowsay_t;

gen_require(`


        type xcowsay_t;
        type xcowsay_exec_t;
        type staff_t;
        type fs_t;
        role staff_r;

')



domtrans_pattern(staff_t, xcowsay_exec_t, xcowsay_t)


#============= xcowsay_t ==============
allow xcowsay_t staff_t:unix_stream_socket connectto;
allow xcowsay_t fs_t:filesystem getattr;

```
<br><br><br><br>


***Step 3.*** Let's work on the ```"fc"``` file now:

```
/usr/bin/xcowsay -- gen_context(staff_u:object_r:xcowsay_exec_t, s0)

```
<br><br><br><br>


***Step 4.*** Make and install the policy module

``` 
sudo make -f /usr/share/selinux/devel/Makefile xcowsay.pp
sudo semodule -i xcowsay.pp
    
```
<br><br><br><br>


***Step 5.*** Change the policy package into CIL format

```
cat xcowsay.pp | /usr/libexec/selinux/hll/pp > xcowsay.cil
cat xcowsay.cil
```
<br>

```
(type xcowsay_t)
(roletype object_r xcowsay_t)
(type xcowsay_exec_t)
(roletype object_r xcowsay_exec_t)
(roleattributeset cil_gen_require system_r)
(roleattributeset cil_gen_require staff_r)
(roletype staff_r xcowsay_t)
(typeattributeset cil_gen_require xcowsay_t)
(typeattributeset cil_gen_require xcowsay_exec_t)
(typeattributeset cil_gen_require application_domain_type)
(typeattributeset application_domain_type (xcowsay_t ))
(typeattributeset cil_gen_require domain)
(typeattributeset domain (xcowsay_t ))
(typeattributeset cil_gen_require corenet_unlabeled_type)
(typeattributeset corenet_unlabeled_type (xcowsay_t ))
(typeattributeset cil_gen_require application_exec_type)
(typeattributeset application_exec_type (xcowsay_exec_t ))
(typeattributeset cil_gen_require exec_type)
(typeattributeset exec_type (xcowsay_exec_t ))
(typeattributeset cil_gen_require file_type)
(typeattributeset file_type (xcowsay_exec_t ))
(typeattributeset cil_gen_require non_security_file_type)
(typeattributeset non_security_file_type (xcowsay_exec_t ))
(typeattributeset cil_gen_require non_auth_file_type)
(typeattributeset non_auth_file_type (xcowsay_exec_t ))
(typeattributeset cil_gen_require entry_type)
(typeattributeset entry_type (xcowsay_exec_t ))
(typeattributeset cil_gen_require staff_t)
(typeattributeset cil_gen_require fs_t)
(allow xcowsay_t xcowsay_exec_t (file (entrypoint)))
(allow xcowsay_t xcowsay_exec_t (file (ioctl read getattr lock map execute open)))
(allow staff_t xcowsay_exec_t (file (ioctl read getattr map execute open execute_no_trans)))
(allow staff_t xcowsay_t (process (transition)))
(typetransition staff_t xcowsay_exec_t process xcowsay_t)
(allow xcowsay_t staff_t (fd (use)))
(allow xcowsay_t staff_t (fifo_file (ioctl read write getattr lock append)))
(allow xcowsay_t staff_t (process (sigchld)))
(allow xcowsay_t staff_t (unix_stream_socket (connectto)))
(allow xcowsay_t fs_t (filesystem (getattr)))
(filecon "/usr/bin/xcowsay" file (staff_u object_r xcowsay_exec_t ((s0) (s0))))

```

<br><br><br><br>


***Step 6.*** Let's try to cut all the unecessary bloat that we have in this .cil file now. For instance we don't need those cil_gen_require statements at all--- unlike Reference Policy where you have to have require statements in order for your policy to build and work. I also added some spacing so you can read it better.

```
(type xcowsay_t)
(roletype object_r xcowsay_t)

(type xcowsay_exec_t)
(roletype object_r xcowsay_exec_t)


(roletype staff_r xcowsay_t)

(typeattributeset application_domain_type (xcowsay_t))
(typeattributeset application_exec_type (xcowsay_exec_t))
(typeattributeset domain (xcowsay_t))
(typeattributeset entry_type (xcowsay_exec_t))


(allow xcowsay_t xcowsay_exec_t (file (entrypoint ioctl read getattr lock map execute open)))
(allow staff_t xcowsay_exec_t (file (ioctl read getattr map execute open execute_no_trans)))
(allow staff_t xcowsay_t (process (transition)))
(typetransition staff_t xcowsay_exec_t process xcowsay_t)

(allow xcowsay_t staff_t (fd (use)))
(allow xcowsay_t staff_t (fifo_file (ioctl read write getattr lock append)))
(allow xcowsay_t staff_t (process (sigchld)))
(allow xcowsay_t staff_t (unix_stream_socket (connectto)))
(allow xcowsay_t fs_t (filesystem (getattr)))

(filecon "/usr/bin/xcowsay" file (staff_u object_r xcowsay_exec_t ((s0) (s0))))

```



