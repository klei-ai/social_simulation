

## Overview

This document provides a piece of code using Agent Interaction Simulations to create a reddit social media simulation with AI agents.

## Environment Setup

### Setting up OpenAI API Key

Choose the method that matches your operating system:

```bash
# For Bash shell (Linux, macOS, Git Bash on Windows)
export OPENAI_API_KEY=<your_openai_api_key>

# For Windows Command Prompt
set OPENAI_API_KEY=<your_openai_api_key>

# For Windows PowerShell
$env:OPENAI_API_KEY="<your_openai_api_key>"
```

### Agent Profile Preparation

Download the sample agent profile file:
- File: `user_data_36.json`
- Location: `./data/reddit/user_data_36.json`

This file contains predefined agent personalities, interests, and behavioral patterns.

## Complete Example Code

```python
import asyncio
import os

from dialogo.models import ModelFactory
from dialogo.types import ModelPlatformType, ModelType

import dialogo
from dialogo import (ActionType, LLMAction, ManualAction,
                   generate_reddit_agent_graph)


async def main():
    """
    Main function that demonstrates a complete Dialogo Reddit simulation.
    This example shows how to:
    1. Set up AI agents with specific models
    2. Define available actions for agents
    3. Create and manage a simulation environment
    4. Execute both manual and AI-driven actions
    """
    
    # =================================================================
    # STEP 1: Model Configuration
    # =================================================================
    
    # Create an OpenAI GPT-4o Mini model instance for our agents
    # This model will power the decision-making for all AI agents
    openai_model = ModelFactory.create(
        model_platform=ModelPlatformType.OPENAI,  # Use OpenAI as provider
        model_type=ModelType.GPT_4O_MINI,         # Specific model variant
    )
    
    # =================================================================
    # STEP 2: Action Space Definition
    # =================================================================
    
    # Define all possible actions that agents can perform in the simulation
    # This creates a rich interaction environment similar to real Reddit
    available_actions = [
        # Post interactions
        ActionType.LIKE_POST,        # Upvote posts
        ActionType.DISLIKE_POST,     # Downvote posts
        ActionType.CREATE_POST,      # Create new posts
        
        # Comment interactions
        ActionType.CREATE_COMMENT,   # Reply to posts
        ActionType.LIKE_COMMENT,     # Upvote comments
        ActionType.DISLIKE_COMMENT,  # Downvote comments
        
        # Discovery and navigation
        ActionType.SEARCH_POSTS,     # Search for specific content
        ActionType.SEARCH_USER,      # Find other users
        ActionType.TREND,            # View trending content
        ActionType.REFRESH,          # Refresh feed
        
        # Social interactions
        ActionType.FOLLOW,           # Follow other users
        ActionType.MUTE,            # Mute users
        
        # Passive action
        ActionType.DO_NOTHING,       # Agent chooses not to act
    ]
    
    # =================================================================
    # STEP 3: Agent Graph Generation
    # =================================================================
    
    # Generate a network of agents based on the profile data
    # This creates the social graph with relationships and personalities
    agent_graph = await generate_reddit_agent_graph(
        profile_path="./data/reddit/user_data_36.json",  # Path to agent profiles
        model=openai_model,                               # AI model for agents
        available_actions=available_actions,              # Actions agents can take
    )
    
    # =================================================================
    # STEP 4: Database and Environment Setup
    # =================================================================
    
    # Define database path for storing simulation data
    db_path = "./data/reddit_simulation.db"
    
    # Clean slate: remove existing database for fresh simulation
    if os.path.exists(db_path):
        os.remove(db_path)
        print("🗑️  Removed existing database for fresh start")
    
    # Create the simulation environment
    # This sets up the virtual Reddit platform with our agents
    env = dialogo.make(
        agent_graph=agent_graph,                        # Our network of agents
        platform=dialogo.DefaultPlatformType.REDDIT,     # Reddit-like platform
        database_path=db_path,                          # Data storage location
    )
    
    # Initialize the environment (creates tables, sets up initial state)
    await env.reset()
    print("🏝️  Dialogo environment initialized successfully!")
    
    # =================================================================
    # STEP 5: Manual Actions (Simulation Kickstart)
    # =================================================================
    
    # Create initial content through manual actions
    # This provides seed content for agents to interact with
    actions_1 = {}
    
    # Agent 0 actions: Create a post and comment on it
    actions_1[env.agent_graph.get_agent(0)] = [
        # Create an introductory post
        ManualAction(
            action_type=ActionType.CREATE_POST,
            action_args={"content": "Hello, world!"}
        ),
        # Comment on the post they just created
        ManualAction(
            action_type=ActionType.CREATE_COMMENT,
            action_args={
                "post_id": "1",                           # ID of the post to comment on
                "content": "Welcome to the Dialogo World!"  # Comment content
            }
        )
    ]
    
    # Agent 1 action: Respond to the post
    actions_1[env.agent_graph.get_agent(1)] = ManualAction(
        action_type=ActionType.CREATE_COMMENT,
        action_args={
            "post_id": "1",                          # Same post as above
            "content": "I like the Dialogo world."     # Positive response
        }
    )
    
    # Execute the manual actions
    print("🎬 Executing manual actions to seed the simulation...")
    await env.step(actions_1)
    
    # =================================================================
    # STEP 6: AI-Driven Actions
    # =================================================================
    
    # Now let all agents make their own decisions using AI
    # LLMAction() allows agents to choose actions based on their personality
    # and the current state of the simulation
    actions_2 = {
        agent: LLMAction()  # Each agent decides what to do autonomously
        for _, agent in env.agent_graph.get_agents()  # For all agents in the graph
    }
    
    print("🤖 Letting agents make autonomous decisions...")
    # Execute AI-driven actions
    await env.step(actions_2)
    
    # =================================================================
    # STEP 7: Cleanup
    # =================================================================
    
    # Properly close the environment and save all data
    await env.close()
    print("✅ Simulation completed successfully!")
    print(f"📊 Simulation data saved to: {db_path}")


# =================================================================
# EXECUTION
# =================================================================

if __name__ == "__main__":
    """
    Entry point for the simulation.
    Uses asyncio to handle the asynchronous nature of the simulation.
    """
    print("🚀 Starting Dialogo Reddit Simulation...")
    asyncio.run(main())
```

## Understanding the Code Structure

### 1. Model Configuration
The simulation uses OpenAI's GPT-4o Mini model to power agent decision-making. Each agent uses this model to:
- Analyze the current state of the platform
- Make decisions based on their personality profile
- Generate realistic content and responses

### 2. Action Types Explained

| Category | Actions | Description |
|----------|---------|-------------|
| **Content Creation** | `CREATE_POST`, `CREATE_COMMENT` | Generate original content |
| **Content Interaction** | `LIKE_POST`, `DISLIKE_POST`, `LIKE_COMMENT`, `DISLIKE_COMMENT` | React to existing content |
| **Discovery** | `SEARCH_POSTS`, `SEARCH_USER`, `TREND`, `REFRESH` | Find and explore content |
| **Social** | `FOLLOW`, `MUTE` | Manage social connections |
| **Passive** | `DO_NOTHING` | Choose not to take action |

### 3. Agent Graph
The agent graph represents:
- **Nodes**: Individual agents with unique personalities
- **Edges**: Relationships between agents (followers, friends, etc.)
- **Attributes**: Agent characteristics (interests, behavior patterns, demographics)

### 4. Two-Phase Execution

#### Phase 1: Manual Actions
- Provides seed content for the simulation
- Ensures agents have something to interact with
- Demonstrates how to inject specific scenarios

#### Phase 2: Autonomous Actions
- Agents make independent decisions
- Realistic emergent behavior occurs
- Social dynamics develop naturally

## Data Storage and Analysis

All simulation data is stored in an SQLite database (`reddit_simulation.db`) containing:
- User profiles and relationships
- Posts and comments with timestamps
- Interaction history (likes, follows, etc.)
- Platform statistics and metrics

## Advanced Usage Patterns

### Multiple Simulation Rounds
```python
# Run multiple rounds of autonomous behavior
for round_num in range(10):
    actions = {
        agent: LLMAction() 
        for _, agent in env.agent_graph.get_agents()
    }
    await env.step(actions)
    print(f"Completed round {round_num + 1}")
```

### Mixed Action Types
```python
# Combine manual and autonomous actions in one step
mixed_actions = {}
mixed_actions[agent_1] = ManualAction(...)  # Specific action
mixed_actions[agent_2] = LLMAction()        # AI decision
await env.step(mixed_actions)
```

### Conditional Actions
```python
# Agents can have different action sets based on conditions
for agent_id, agent in env.agent_graph.get_agents():
    if agent.profile.get("age") > 25:
        actions[agent] = LLMAction()  # Full action set
    else:
        # Restricted actions for younger agents
        actions[agent] = ManualAction(ActionType.REFRESH)
```










---
