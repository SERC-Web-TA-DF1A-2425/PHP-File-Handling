# Codespaces Development Environment and Usage

## Development Environment

This repository is configured to run in a GitHub Codespace. A Codespace is a cloud-hosted development environment that can be accessed directly in a web browser or in Visual Studio Code. This allows for a consistent development environment across different machines and operating systems.

### Services Installed

- **PHP**: PHP (8.2) is installed for running web applications.
- **MariaDB**: A MySQL-compatible database server for storing data. This is installed as a docker container.
- **phpMyAdmin**: A web-based database management tool for managing MariaDB. This is installed as a docker container.

### Port Forwarding

Ports are forwarded from the Codespace to the host machine to allow access to services running in the Codespace outside of the Codespace and VS Code interface. For example, this allows for a web application to be accessed in a web browser on the local machine.

The following ports are forwarded for use in the Codespace. If running the codespace in a web browser, these will be forwarded via a reverse proxy. If running the codespace in a local development environment or directly in VS Code, these ports will be forwarded to the host machine and be available at `localhost`.

- **8080**: PHP's built-in web server. This is not running by default but can be started with the command: `php -S localhost:8080`. This command is also pre-configured as a VS Code task.
- **3306**: MariaDB database server.
- **8081**: phpMyAdmin database management tool.

### VS Code Extensions

The following extensions are installed in the Codespace:

- **PHP Intelephense**: Provides intelligent code completion and analysis for PHP.
- **PHP Debug**: PHP debugging support.
- **SQLTools**: SQL database management tool.
- **SQLTools Driver for MySQL/MariaDB**: Adds support for MySQL/MariaDB databases to SQLTools.

## How to Use the Codespace

To use the Codespace, follow these steps:

1. **Open the Repository in Codespaces**:
   - Navigate to the repository on GitHub.
   - Click the `Code` button and select `Open with Codespaces`.
   - Create a new Codespace or open an existing one.

2. **Start the Development Environment**:
   - The Codespace will automatically build the development container.
   - Wait for the setup to complete (this may take a few minutes).

3. **Run Your Application**:
   - Use the terminal to start your PHP application (e.g., `php -S localhost:8000`).
   - Access the application using the forwarded port in the Codespace.

4. **Install Dependencies**:
   - Run `composer install` to install PHP dependencies.

5. **Debugging**:
   - Use the built-in debugger in VS Code for PHP to set breakpoints and debug your application.

6. **Stop the Codespace**:
   - When done, stop the Codespace to save resources.

### Additional Notes

- Codespaces contain persistent storage, so any changes made in the Codespace will be saved for when the Codespace is re-started. However, this persistent storage will be deleted after 30 days of inactivity.
- It is recommended to commit changes to the repository to ensure that changes are not lost. This can be done directly in the Codespace or in the GitHub interface.

## Further Reading

- [GitHub Codespaces Documentation](https://docs.github.com/en/codespaces)
- [VS Code Remote Development](https://code.visualstudio.com/docs/remote/remote-overview)
