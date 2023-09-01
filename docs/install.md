# Installing MethBat
## From GitHub
Conda updates usually lag the GitHub release by a couple days.
Use the following instructions to get the most recent version directly from GitHub:

1. Navigate to the [latest release](https://github.com/PacificBiosciences/MethBat/releases/latest) and download the tarball file (e.g. `methbat-{version}-x86_64-unknown-linux-gnu.tar.gz`).
2. Decompress the tar file.
3. (Optional) Verify the md5 checksum.
4. Test the binary file by running it with the help option (`-h`).
5. Visit the [User guide](./user_guide.md) for details on running MethBat.

### Example with v0.8.3
```bash
# modify this to update the version
VERSION="v0.8.3"
# get the release file
wget https://github.com/PacificBiosciences/MethBat/releases/download/${VERSION}/methbat-${VERSION}-x86_64-unknown-linux-gnu.tar.gz
# decompress the file into folder ${VERSION}
tar -xzvf methbat-${VERSION}-x86_64-unknown-linux-gnu.tar.gz
cd methbat-${VERSION}-x86_64-unknown-linux-gnu
# optional, check the md5 sum
md5sum -c methbat.md5
# execute help instructions
./methbat -h
```
