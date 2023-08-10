# Pipeline Power Tool

## Calculator

To run the Pipeline Power Tool Calculator:
1. Make sure the database and API are running
2. Update the URL in the DemoSection with the URL of the API
3. Create a docker image using the Dockerfile in root
4. Start the container

### Example Docker-compose
```dockerfile
  website:
    image: ${IMAGE_NAME}
    build:
      context: .
    container_name: calculator
    restart: always
    ports:
      - "8083:80"
```

