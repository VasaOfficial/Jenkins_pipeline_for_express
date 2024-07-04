# Use the official Jenkins image as the base image
FROM jenkins/jenkins:lts

# Set environment variables
ENV JAVA_OPTS -Djenkins.install.runSetupWizard=false

# Install necessary packages (optional)
USER root
RUN apt-get update

VOLUME /var/jenkins_home

# Switch back to the jenkins user
USER jenkins

# Expose ports
EXPOSE 8080