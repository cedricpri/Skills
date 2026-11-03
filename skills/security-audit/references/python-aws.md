# Python / AWS / data security checklist

## Code
- Deserialization of untrusted files: `pickle`, `joblib.load`, `torch.load` without `weights_only=True`, `yaml.load` without `SafeLoader`, `numpy.load(allow_pickle=True)`.
- `subprocess` with `shell=True` or user input in the command, `os.system`, `eval`, `exec`.
- SQL built with f-strings or `%` formatting instead of parameters.
- Paths built from input (uploaded filenames, S3 keys) without normalization: traversal with `../`.
- Archives extracted without checking member paths (`tarfile` before the `filter=` argument, `zipfile`).
- Requests to user-supplied URLs (SSRF), `verify=False` on HTTPS requests.
- Temporary files with predictable names; files written world-readable.

## LLM and ASR pipelines
- Personal or health data sent to external APIs: is it allowed, minimized, and is the provider's retention acceptable?
- Prompt injection: untrusted text (transcripts, documents, emails) that can change instructions, especially when the model output triggers actions or is shown to other users.
- Model output used in SQL, shell, file paths or HTML without validation.

## Streamlit and internal apps
- Authentication present and enforced on every page; credentials not in the repo (`secrets.toml` ignored by git).
- Apps bound to `0.0.0.0` or exposed publicly without need.
- File uploaders validating type and size.

## AWS
- No access keys in code, notebooks, `.env` files committed, or CI logs; prefer IAM roles.
- IAM policies with `*` actions or resources where narrower ones work; Lambda roles limited to the buckets and services they use.
- S3 buckets: Block Public Access on, bucket policies not granting public or cross-account access unintentionally, encryption on, versioning for important data.
- Presigned URLs with short expiry and for specific keys.
- Lambda environment variables holding secrets (prefer Secrets Manager or SSM Parameter Store); function URLs without auth.
- CloudWatch logs containing personal data, with retention set.
- Security groups open to `0.0.0.0/0` on non-public ports.

## Personal and health data
- Member ids, names, national numbers, medical or reimbursement data in logs, exceptions, notebook outputs, test fixtures, sample files committed to git, or copied to places with weaker access control.
- Data kept longer than needed; local copies on laptops.

## Dependencies
- `pip-audit` findings, unpinned dependencies, packages installed from unusual indexes.
