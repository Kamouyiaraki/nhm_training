# Data storage

- Home folders are found in `/home`
- Important project data should be stored in `/workspaces/groups`
- For temporary storage needed during job execution use `/mbl/share/scratch`
- Shared software for use by anyone can be stored in `/software`  

## Home folders

- **Path**: `/home/<username>`
- **Backed up**: once a day
- **Quota**: 50GB
- **Used for**:

  - Small files
  - Software only accessible to you (personal Conda environments, etc.)

When you log in, you'll be in your home directory, a personal area only you can access. Here, you can save small files and install software (Conda environments, etc.), but space is limited to 30GB. For larger files or long-term projects, use [group/project](#groupproject-folders) folders.

## Group/project folders

- **Paths**:<br>
The groups folder can be accessed via any of these symlinks:
  - `/workspaces/groups`
  - `/mbl/share/workspaces/groups`   
  - `/gpfs/nhmfsa/bulk/share/data/mbl/share/workspaces/groups`

- **Backed up**: every 1-2 days
- **Quota**: 2TB
- **Used for**:

  + Projects
  + Collaboration with colleagues
  + Large files
  + Important data intended for long-term storage

It's best to store your important work here. If you need a new folder for your project, reach out to TS for assistance. Colleagues can also be given access, making these group workspaces ideal for collaborative efforts. Each folder has a default storage limit of 2TB, but can be increased if necessary.

## Shared software

- **Paths**:

  - `/software`   
  - `/mbl/share/software`  
  - `/gpfs/nhmfsa/bulk/share/data/mbl/share/software`

- **Backed up**: every 1-2 days
- **Quota**: None
- **Used for**:

  - Software available to all users
  - Software you have compiled (usually with the `make` command) and want to make available for other users

## Scratch space

- **Path**: `/mbl/share/scratch`
- **Backed up**: never
- **Quota**: None
- **Used for**:

  - Temporary storage for files during job execution
  - Data that can be easily recreated

Scratch space serves as a temporary storage area for files needed only during job execution. The benefit of using this folder is that it won't affect your overall disk quota. Files in this folder are automatically deleted after 21 days of inactivity. However, it's recommended to delete them manually once you're finished.

> Refrain from using the `/tmp` directory for temporary output, as it is reserved for system processes. Any data stored in `/tmp` may be deleted without notice.
