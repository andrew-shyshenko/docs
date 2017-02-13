#####Overview

Distributed Virtual Router (DVR) - 


#####FAQ

* How many **public** IP does DVR require without any FIP for VM?

    1 for snat router port in controller (network) node
    1 for fip router port in each compute node (this is not fip for VM)
     