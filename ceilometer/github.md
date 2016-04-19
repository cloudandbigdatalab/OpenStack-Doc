##A Short Guide on OpenStack Fixes##

There are many different ways of conducting a bug fix on OpenStack. This is a short point by point guide on how conduct them with commands below.

1. Create a new instance or virtual machine to guarentee a clean work space.  This can be done in OpenStack, Vagrant, or VirtualBox

  1. In Vagrant:
  ```
  Vagrant up
  ```
  2. CLI Openstack:
  ```
  openstack server create --nic net-id=NETWORK_ID \
          	              --flavor IMAGE_FLAVOR \
          	              --key-name KEYPAIR_NAME \
          	              --image MY_IMAGE_NAME \
          	              VM_NAME
  ```

2. Once the instance is up.  You'll need to install some basic packages:
   ```
   apt-get -y update git maven python-dev
   ```

3. Configure your git credentials:
  ```
  git config --global user.name "First Last"
  git config --global user.email First.Last@gmail.com
  git config --global gitreview.username MyAwesomeUserName
  git config --global core.editor vi
  ```
4. Copy your garrett ssh key over
5. FIX THE BUG!!
6. Checkout of git

  ```
  git review -s -v
  git checkout -b bug/1234567
  git add .
  git status
  git commit       #Closes-Bug: #1234567
  git status
  ```
  
  And if you forgot to add the close bug:
  ```
  git commit --amend  (if you forgot to add the close bug tag..)
  ```
7. When complete:

  ```
  git branch -d bug/1234567
  ```
