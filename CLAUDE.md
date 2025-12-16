# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Context

This repository is `springai-mcp-rag-dev`, focused on integrating Model Context Protocol (MCP) with Spring AI for Retrieval-Augmented Generation (RAG) applications.

## Project Structure

The repository currently contains:
- `mcp-rag-quiz-game.html` - An interactive HTML5 game for learning MCP and RAG concepts
- No build configuration files (package.json, pom.xml, etc.) have been detected yet

## Development Setup

Since this appears to be the early stage of a Spring AI + MCP + RAG project, when setting up the development environment:

1. **For Java/Spring development**:
   - Check for `pom.xml` or `build.gradle` in the project directory
   - Look for Spring Boot application files (`@SpringBootApplication`)
   - Common commands: `mvn spring-boot:run` or `./gradlew bootRun`

2. **For the HTML game**:
   - The game runs standalone in any modern browser
   - Open `mcp-rag-quiz-game.html` directly

## Architecture Notes

### Expected Components for Spring AI + MCP + RAG:

1. **MCP Integration Layer**:
   - MCP server implementation for tool exposure
   - Client connectors for AI model interaction
   - Protocol handlers for JSON-RPC communication

2. **RAG Components**:
   - Vector store integration (e.g., ChromaDB, Pinecone, pgvector)
   - Document processors and chunking strategies
   - Embedding service configuration

3. **Spring AI Core**:
   - Model providers configuration
   - Prompt templates and chains
   - Memory and conversation management

## Key Technologies

- **Spring AI**: Framework for AI application development in Java
- **MCP (Model Context Protocol)**: Anthropic's protocol for AI-external system communication
- **RAG (Retrieval-Augmented Generation)**: Technique for enhancing AI responses with external knowledge

## Important Considerations

- When implementing MCP servers, ensure proper error handling for JSON-RPC
- RAG implementations should consider chunk size optimization for your use case
- Vector similarity thresholds need tuning based on your domain
- Spring AI configurations typically reside in `application.yml` or `application.properties`

## Common Tasks

When the Spring project structure is established, typical commands include:

- **Build**: `mvn clean install` or `./gradlew build`
- **Test**: `mvn test` or `./gradlew test`
- **Run**: `mvn spring-boot:run` or `./gradlew bootRun`
- **Package**: `mvn package` or `./gradlew bootJar`

For development, ensure you have:
- Java 17+ installed
- Maven or Gradle configured
- IDE with Spring support (IntelliJ IDEA, Eclipse, VS Code with Spring extensions)