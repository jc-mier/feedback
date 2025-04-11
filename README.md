## Feedback

Feedback Management System

### Load custom apps through apps.json file

Base64 encoded string of `apps.json` file needs to be passed in as build arg environment variable.

Create the following `apps.json` file:

```json
[
  {
    "url": "https://github.com/frappe/erpnext",
    "branch": "version-15"
  },
  {
    "url": "https://github.com/frappe/hrms",
    "branch": "version-15"
  },
  {
    "url": "https://github.com/jc-mier/feedback.git",
    "branch": "main"
  }
]
```

Note:

- The `url` needs to be http(s) git url with personal access tokens without username eg:- public repo `http://{{PAT}}@github.com/project/repository.git` for private repo(`https://ght***************@git.example.com/project/repository.git`) in case of private repo.
- Add dependencies manually in `apps.json` e.g. add `erpnext` if you are installing `hrms`.
- Use fork repo or branch for ERPNext in case you need to use your fork or test a PR.

In windows 11: 
- create frappe folder
- then enter `wsl` to enter linux file system to base64 string from json file
  
```shell
mkdir frappe 
cd frappe

wsl
```

In linux/unix Generate base64 string from json file:

```shell
export APPS_JSON_BASE64=$(base64 -w 0 /path/to/apps.json)
```


Test the Previous Step: Decode the Base64-encoded Environment Variable

To verify the previous step, decode the `APPS_JSON_BASE64` environment variable (which is Base64-encoded) into a JSON file. Follow the steps below:

1. Use the following command to decode and save the output into a JSON file named apps-test-output.json:

```shell
echo -n ${APPS_JSON_BASE64} | base64 -d > apps-test-output.json
```

2. Open the apps-test-output.json file to review the JSON output and ensure that the content is correct.

### Clone frappe_docker and switch directory

```shell
git clone https://github.com/frappe/frappe_docker
cd frappe_docker
```

### Configure build

Common build args.

- `FRAPPE_PATH`, customize the source repo for frappe framework. Defaults to `https://github.com/frappe/frappe`
- `FRAPPE_BRANCH`, customize the source repo branch for frappe framework. Defaults to `version-15`.
- `APPS_JSON_BASE64`, correct base64 encoded JSON string generated from `apps.json` file.

Notes

- Use `buildah` or `docker` as per your setup.
- Make sure `APPS_JSON_BASE64` variable has correct base64 encoded JSON string. It is consumed as build arg, base64 encoding ensures it to be friendly with environment variables. Use `jq empty apps.json` to validate `apps.json` file.
- Make sure the `--tag` is valid image name that will be pushed to registry. See section [below](#use-images) for remarks about its use.
- `.git` directories for all apps are removed from the image.


### Custom build image

This method builds the base and build layer every time, it allows to customize Python and NodeJS runtime versions. It takes more time to build.

It uses `images/custom/Containerfile`.

```shell
docker build \
  --build-arg=FRAPPE_PATH=https://github.com/frappe/frappe \
  --build-arg=FRAPPE_BRANCH=version-15 \
  --build-arg=PYTHON_VERSION=3.11.9 \
  --build-arg=NODE_VERSION=18.20.2 \
  --build-arg=APPS_JSON_BASE64=$APPS_JSON_BASE64 \
  --tag=fc \
  --file=images/custom/Containerfile .
```

Replace copy fc.yml file to frappe_docker folder then execute

`docker compose -f fc.yml up -d`


## Final steps

Wait for 5 minutes for ERPNext site to be created or check create-site container logs before opening browser on port 8080. (username: Administrator, password: admin)

If you ran in a Dev Docker environment, to view container logs: docker compose -f fc.yml logs -f create-site. Don't worry about some of the initial error messages, some services take a while to become ready, and then they go away.
