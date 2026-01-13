This is how the AllTheBacteria data was downloaded to anatra

## Lexicmap Indexes.

**Step1. Create conda env and install awscli**

```
conda create --prefix /mnt/lifesciences/pra-0066/users/ajrh20_alex/envs/aws -c conda-forge awscli
```

**Step2. Test aws** 

```
aws s3 ls s3://allthebacteria-lexicmap/202408/ --no-sign-request
```

**Step3. Download index in tmux**

```
tmux new -s lexicmap-index
aws s3 cp s3://allthebacteria-lexicmap/202408/ atb.lmi --recursive --no-sign-request
```

## sketchlib indexes
Downloaded the sketch data (skd) and metadata (skm), aggregated release (202408) files from: https://osf.io/rceq5/files/osfstorage 


## ATB Assemblies
```
This should be release 0.2 + incremental release 2024-08
```
Step 1. First download the filelist
```
wget https://osf.io/download/4yv85/ -O file_list.all.latest.tsv.gz
gunzip file_list.all.latest.tsv.gz 
```
Step2. Extract tarballs and their URLs, informming awk the file is tab deliminated. 
```
cat file_list.all.latest.tsv | awk -F'\t' 'NR>1 {print $5"\t"$6}' | uniq > tar2url_update.tsv
```
Step 3. Allot of the tar urls are duplicated in tar2url_updated.tsv as assemblies are provided in batches of xzipped tar archives. Deduplicate.
```
sort -u tar2url_update.tsv > unique_tarurls.tsv
```
Step 4. Now download assemblies from OSF on login node in tmux session, as the compute node has no internet access. Script lives in getdata_loginnode_rep.sh
```
#!/bin/bash
output="/mnt/lifesciences/shared-data/AllTheBacteria/genomes/assemblies_download"
log="/mnt/lifesciences/shared-data/AllTheBacteria/genomes/wget_download.log"
err="/mnt/lifesciences/shared-data/AllTheBacteria/genomes/wget_download.err"
mkdir -p "$output"
while IFS=$'\t' read -r name url; do
  echo "$(date) downloading $name" >> "$log"
  wget -O "$output/$name" "$url"
done < unique_tarurls.tsv (edited) 
```
