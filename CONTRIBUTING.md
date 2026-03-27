# Contributing to PropChain Smart Contracts

Thank you for your interest in contributing to PropChain Smart Contracts! This guide will help you get started with contributing to our Rust-based smart contract system.

> **Documentation Version**: 2.0.0 — Updated March 2026

## 🚀 Getting Started

### Prerequisites

Before you start contributing, make sure you have:

- **Rust** 1.75+ installed with stable toolchain (pinned in `rust-toolchain.toml`)
- **cargo-contract** 3.x CLI for ink! smart contract development
- **Git** for version control
- Basic understanding of **Rust** and **ink!** framework
- Familiarity with **Substrate/Polkadot** ecosystem

### Setup

1. **Fork the repository**
   ```bash
   # Fork on GitHub, then clone your fork
   git clone https://github.com/YOUR_USERNAME/PropChain-contract.git
   cd PropChain-contract
   ```

2. **Install development tools**
   ```bash
   # Install Rust (if not already installed)
   curl https://sh.rustup.rs -sSf | sh

   # Install cargo-contract
   cargo install cargo-contract --locked --version "^3"

   # Add WASM target
   rustup target add wasm32-unknown-unknown

   # Optional: dependency auditing tools
   cargo install cargo-deny cargo-audit
   ```

3. **Set up your development environment**
   ```bash
   # Install pre-commit hooks (required — see Code Style section)
   ./scripts/setup-pre-commit.sh

   # Build the contracts
   cargo contract build

   # Run tests to ensure everything works
   cargo test
   ```

## 📋 Contribution Types

We welcome various types of contributions:

### 🐛 Bug Reports
- Use the [Bug Report Template](.github/ISSUE_TEMPLATE/bug_report.md)
- Include detailed reproduction steps
- Provide environment details (Rust version, OS, etc.)
- Add relevant logs or error messages

### 💡 Feature Requests
- Use the [Feature Request Template](.github/ISSUE_TEMPLATE/feature_request.md)
- Describe the use case and motivation
- Consider the impact on existing functionality
- Suggest implementation approach if possible

### 🔧 Code Contributions
- **Bug fixes** - Resolve existing issues
- **New features** - Implement approved functionality
- **Documentation** - Improve code comments and docs
- **Tests** - Add or improve test coverage
- **Optimizations** - Performance improvements

## 🛠️ Development Workflow

### 1. Create a Branch
```bash
# Create a feature branch from main
git checkout -b feature/your-feature-name

# Or for bug fixes
git checkout -b fix/issue-number-description
```

### 2. Make Changes
- Follow our [Code Style Guidelines](#code-style)
- Write tests for new functionality
- Update documentation as needed
- Ensure all tests pass

### 3. Test Your Changes
```bash
# Run all tests
cargo test

# Run contract-specific tests
cargo contract test

# Build in release mode to check for warnings
cargo contract build --release

# Run clippy for linting
cargo clippy -- -D warnings

# Format code
cargo fmt
```

### 4. Commit Your Changes
```bash
# Stage your changes
git add .

# Commit with descriptive message
git commit -m "feat: add property tokenization feature

- Implement NFT minting for real estate properties
- Add metadata storage for property details
- Include comprehensive unit tests

Fixes #123"
```

### 5. Push and Create PR
```bash
# Push to your fork
git push origin feature/your-feature-name

# Create Pull Request on GitHub
```

## 📝 Code Style Guidelines

See [docs/code-style-guide.md](docs/code-style-guide.md) for the full reference. The highlights are:

### Automated Enforcement

All style checks run automatically via pre-commit hooks (installed by `./scripts/setup-pre-commit.sh`).  
You can also run them manually:

```bash
# Format
cargo fmt

# Lint (zero warnings policy)
cargo clippy -- -D warnings

# Run all pre-commit checks
pre-commit run --all-files
```

### Rust Standards
- Follow [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)
- `rustfmt.toml` enforces formatting — never work around it
- `clippy.toml` enforces lint thresholds — fix warnings, don't silence them with `#[allow]` without a comment justifying why
- Prefer `unwrap()` only inside `#[cfg(test)]` blocks

### ink! Smart Contract Best Practices
- Keep contract logic simple and gas-efficient
- Use proper error handling with `Result` types
- Implement proper access control with `#[ink(message)]` modifiers
- Add comprehensive documentation for public functions
- Consider storage optimization for on-chain data

### Naming Conventions
```rust
// Contract names: PascalCase
pub struct PropertyRegistry { ... }

// Functions: snake_case
#[ink(message)]
pub fn register_property(&mut self, property_id: AccountId) { ... }

// Types: PascalCase
pub type PropertyId = AccountId;

// Constants: SCREAMING_SNAKE_CASE
pub const MAX_PROPERTIES: u32 = 1000;
```

### Documentation
```rust
/// Registers a new property in the registry.
///
/// # Arguments
///
/// * `property_id` - The unique identifier for the property
/// * `metadata` - Property metadata including location and details
///
/// # Returns
///
/// Returns `Result<(), Error>` indicating success or failure
///
/// # Example
///
/// ```rust
/// let result = contract.register_property(property_id, metadata);
/// assert!(result.is_ok());
/// ```
#[ink(message)]
pub fn register_property(&mut self, property_id: AccountId) -> Result<(), Error> {
    // implementation
}
```

## 🧪 Testing Guidelines

### Unit Tests
- Write tests for all public functions
- Test both success and error cases
- Use descriptive test names
- Mock external dependencies when needed

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_register_property_succeeds() {
        // Arrange
        let mut contract = PropertyRegistry::new();
        let property_id = AccountId::from([0x1; 32]);
        
        // Act
        let result = contract.register_property(property_id);
        
        // Assert
        assert!(result.is_ok());
    }
}
```

### Integration Tests
- Test contract interactions
- Verify gas usage is reasonable
- Test edge cases and boundary conditions

## 📋 Pull Request Process

### Before Submitting
- [ ] All tests pass (`cargo test`)
- [ ] Code is formatted (`cargo fmt`)
- [ ] No clippy warnings (`cargo clippy -- -D warnings`)
- [ ] Pre-commit hooks pass (`pre-commit run --all-files`)
- [ ] Documentation updated for any changed behaviour (see [Documentation Guidelines](#-documentation-guidelines))
- [ ] `CHANGELOG.md` updated (if applicable)
- [ ] `cargo audit` passes — no new CVEs (`cargo audit --deny warnings`)
- [ ] `cargo deny check` passes

### PR Description
Your PR should include:
- **Title**: Clear and descriptive (Conventional Commits format: `feat:`, `fix:`, `docs:`, etc.)
- **Description**: What changes were made and why
- **Testing**: How you tested the changes
- **Breaking Changes**: Clearly highlight any breaking changes
- **Documentation**: Links to updated docs

### Review Process
1. **Automated Checks**: CI pipeline runs tests, linting, security scan, and cargo-audit
2. **Peer Review**: At least one maintainer must review and approve
3. **Security Review**: Add `security-review` label for changes to auth, crypto, or cross-contract calls
4. **Approval**: Merge after all checks pass and approval is given

## 🔒 Security Considerations

- Never commit private keys or sensitive data
- Follow secure coding practices for smart contracts — see [SECURITY.md](SECURITY.md)
- Consider gas optimization and DoS protection
- Report security vulnerabilities privately via the process in [SECURITY.md](SECURITY.md)

## 📚 Documentation Guidelines

Documentation must stay in sync with code changes. When you submit a PR:

1. **Update docs**: If you change contract behaviour, update the corresponding file in `docs/`.
2. **New features**: Create or update `docs/` coverage. All public `#[ink(message)]` functions must have rustdoc.
3. **Version the doc**: Bump the `> **Documentation Version**` header in any doc file you edit.
4. **Architecture decisions**: If your change affects system design, add an ADR in `docs/adr/`.
5. **Tutorials**: If you add a major new capability, add a tutorial in `docs/tutorials/`.

See [docs/documentation-maintenance.md](docs/documentation-maintenance.md) for the full process.

## 📚 Resources

### Development Tools
- [ink! Documentation](https://use.ink/)
- [Substrate Documentation](https://substrate.io/)
- [Rust Book](https://doc.rust-lang.org/book/)
- [Code Style Guide](docs/code-style-guide.md)

### Community
- [Polkadot Discord](https://discord.gg/polkadot)
- [Substrate Stack Exchange](https://substrate.stackexchange.com/)
- [Rust Users Forum](https://users.rust-lang.org/)

## 🏆 Recognition

Contributors are recognized in:
- README.md contributors section
- Release notes for significant contributions
- Annual contributor appreciation post

## 📞 Get Help

If you need help with contributing:
- **Discord**: [PropChain Community](https://discord.gg/propchain)
- **GitHub Issues**: Tag maintainers for questions
- **Email**: dev@propchain.io

## 📄 License

By contributing, you agree that your contributions will be licensed under the same license as the project (MIT License).

---

Thank you for contributing to PropChain Smart Contracts! 🎉

Your contributions help make decentralized real estate a reality.
