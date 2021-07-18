# **Changing-Selinux-Refpolicy-to-CIL-Policy**
<br><br>

I will show you how to change your Selinux Refpolicy into CIL Policy with an example. The example program I have used is called xcowsay and I have written the policy in Refpolicy format and will translate it into CIL for you. You will see how much less policy there is and how easier it is to write.
<br><br>



***Step 1.*** Create your own policy module using the Refpolicy language which should have a ```".te"``` and a ```".fc"``` file. My example looks like the following

```
touch xcowsay.te
touch xcowsay.fc
```
<br><br><br><br>


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
