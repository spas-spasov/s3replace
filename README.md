# s3replace

![Screenshot](/etc/preview.png?raw=true)

Sometimes we have to replace the same thing in lots of files in an S3 bucket - for example, when the paywall code changes or when we switch commenting vendors. This repo is our automated solution.

It uses the AWS API to roll through all of the objects in a bucket:
1. Filtering the objects to search using a regular expression, it downloads any object that matches.
2. Of those objects that match, it uses another regular expression to find the relevant code to replace.
3. If the object's content is a match, you'll be given a preview and asked for confirmation before anything is changed.
4. It replaces the code, copying metadata such as the `ContentType`, `ContentDisposition` and other key fields. A backup of the file is saved locally, just in case.

## Requirements

- Python 3 - in Mac `brew install python` in Debian it is installed by default
- `pipenv` - in Mac `brew install pipenv` in Debian `sudo apt install pipenv`

## Installation and setup

1. Clone this repo
2. Install requirements using `pipenv`.

## Usage

#### Configuration

In _s3replace/__main__.py_:
- update the `needle_pattern` at the top. This pattern will be used by `re.search` to find matching documents and it'll be the content that is replaced using `re.sub`.
- set `replace_with` at the top of the file to the text you want to replace the `needle_pattern` with
- update the `key_pattern` variable to match the keys you want to run `needle_pattern` against; the more specific this is, the better; files that match this won't be downloaded, which is the slowest part of the process
- `needle_pattern_max_count` is the number of times `needle_pattern` should be found in the file before it is replaced, otherwise the file will be skipped, and a copy of it will be saved in `backups/too_many_matches`


#### Running

This runs as a command line tool. See all the options by running `python s3replace --help`:

```
$ python s3replace --help

Find and replace for an S3 bucket.

Usage:
  s3replace <bucket> [--dry-run|--force-replace] [--access-key-id=<key>] [--secret-access-key=<key>]
  s3replace -h | --help
  s3replace --version

Options:
  -h --help                 Show this screen.
  --version                 Show version.
  --dry-run                 Don't replace.
  --access-key-id=<key>     AWS access key ID
  --secret-access-key=<key> AWS secret access key
  --force-replace          Force replace without confirmation.
```

Basic usage only requires a bucket name and credentials:

```sh
$ python s3replace <bucket> --access-key-id=<yourid> --secret-access-key=<yourkey>
```

You can pass your AWS credentials using the flags, as above, or you can provide them using any of the [other methods](http://boto3.readthedocs.io/en/latest/guide/quickstart.html#configuration) supported by `boto3`.

## Get list of the backup files

```bash
find backups/ -type f -printf '%d\t%P\n' | sort -r -nk1 | cut -f2-
```

## Copyright

&copy; 2018 _The Dallas Morning News_
