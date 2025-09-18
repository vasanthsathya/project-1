Cluster formation
=====================

1. In the ``/opt/omnia/input/project_default/omnia_config.yml``, ``/opt/omnia/input/project_default/security_config.yml``, and ``/opt/omnia/input/project_default/storage_config.yml`` files, provide the `required details <../schedulerinputparams.html>`_. For ``/opt/omnia/input/project_default/telemetry_config.yml``, the details can be found `here <../../../../Telemetry/index.html#id13>`_.

2. Create an inventory file in the *omnia* folder. Check out the `sample inventory <../../../samplefiles.html>`_ for more information. If a hostname is used to refer to the target nodes, ensure that the domain name is included in the entry. IP addresses are also accepted in the inventory file.

.. include:: ../../Appendices/hostnamereqs.rst

.. note::
     * Omnia creates log files, available at: ``/opt/omnia/log/``.
     * If only Slurm is being installed on the cluster, docker credentials are not required.


3. ``omnia.yml`` is a wrapper playbook comprising of:

    * ``security.yml``: This playbook sets up centralized authentication (LDAP/FreeIPA) on the cluster. For more information, `click here. <Authentication.html>`_
    * ``storage.yml``: This playbook sets up storage tools like `BeeGFS <Storage/BeeGFS.html>`_ and `NFS <Storage/NFS.html>`_.
    * ``scheduler.yml``: This playbook sets up job schedulers (`Slurm <install_slurm.html>`_ or `Kubernetes <install_kubernetes.html>`_) on the cluster.
    * ``rocm_installation.yml``: This playbook sets up the `ROCm platform for AMD GPU accelerators <AMD_ROCm.html>`_.

To run ``omnia.yml``: ::

        ssh omnia_core
        cd /omnia
        ansible-playbook omnia.yml -i <inventory_file_path>

.. note::
    
    * If you want to view or edit the ``omnia_config.yml`` file, run the following command:

        - ``ansible-vault view omnia_config.yml --vault-password-file .omnia_vault_key`` -- To view the file.

        - ``ansible-vault edit omnia_config.yml --vault-password-file .omnia_vault_key`` -- To edit the file.

    * Use the ansible-vault view or edit commands and not the ansible-vault decrypt or encrypt commands. If you have used the ansible-vault decrypt or encrypt commands, provide 644 permission to the parameter files.

4. Once ``omnia.yml`` playbook is successfully executed, the cluster is up and running with the required application stack.