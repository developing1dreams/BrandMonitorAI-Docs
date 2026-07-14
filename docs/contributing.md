# Contributing

Thank you for contributing to **BrandMonitorAI**, part of the [ZeroShield](https://zeroshield.ai) ecosystem.

## How to contribute

1. Fork the repository (or create a branch if you have write access)
2. Create a focused feature/fix branch
3. Make changes with tests and docs where appropriate
4. Run quality gates (see below)
5. Open a pull request with a clear description and test plan

## Development setup

Follow [installation.md](installation.md) and [local-development.md](local-development.md).

## Quality gates

```bash
npm run lint
npm run typecheck
npm test
npm run build

cd orchestration-backend/api
pytest -q
```

## Pull request guidelines

* Describe **why** the change is needed
* Link related issues
* Note any new environment variables (update `.env.example` + `configuration.md`)
* Include screenshots for UI changes when useful
* Keep secrets out of the PR

## Code of conduct (summary)

* Be respectful and professional
* Assume good intent
* Prefer constructive, specific feedback
* Do not submit malware, exploit payloads, or credentials

## Security disclosures

Please report security vulnerabilities privately to [support@zeroshield.ai](mailto:support@zeroshield.ai) rather than opening a public issue.

## License

By contributing, you agree that your contributions are licensed under the MIT License (see [`../LICENSE`](../LICENSE)).
