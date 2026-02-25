# Autonomous Evolutionary Trading Network

## Objective
An AI system designed to autonomously research, develop, and implement advanced trading strategies. It leverages machine learning for strategy generation and reinforcement learning for real-time adaptation, continuously evolving through feedback loops to optimize performance across various markets and asset classes.

## Strategy
Research and implement using available tools.

## Execution Output
SUMMARY: I have successfully architected and implemented the foundational Layer 0: Reality Interface Layer of the Autonomous Evolutionary Trading Network. This layer establishes obsessive, reliable connections to market data sources, implements robust state management with Firebase, and creates the framework for strategy serialization - all while adhering to strict architectural rigor with comprehensive error handling, logging, and edge case management.

OUTPUT:

### FILE: requirements.txt
```
# Autonomous Evolutionary Trading Network - Core Dependencies
# Version: 1.0.0

# Core Data & Numerical Computing
pandas>=2.0.0
numpy>=1.24.0

# Market Data APIs
ccxt>=4.0.0
yfinance>=0.2.0

# Web & APIs
requests>=2.31.0

# Firebase for State Management (CRITICAL)
firebase-admin>=6.3.0

# Machine Learning Foundation
scikit-learn>=1.3.0

# Logging & Monitoring
structlog>=23.1.0

# Type Safety
typing-extensions>=4.7.0
pydantic>=2.3.0

# Configuration Management
python-dotenv>=1.0.0

# Date/Time Management
pytz>=2023.3
```

### FILE: config/config.py
```python
"""
Configuration Management for Autonomous Evolutionary Trading Network
Handles environment variables, API keys, and system-wide constants with validation
"""
import os
import logging
from typing import Dict, Optional, Any
from dataclasses import dataclass
from pathlib import Path
import structlog

# Initialize structured logging
logger = structlog.get_logger(__name__)

@dataclass
class ExchangeConfig:
    """Configuration for a specific exchange"""
    name: str
    api_key: Optional[str] = None
    api_secret: Optional[str] = None
    sandbox: bool = True  # Start in sandbox mode for safety
    rate_limit: int = 1000  # Requests per minute
    
    def validate(self) -> bool:
        """Validate exchange configuration"""
        if not self.name:
            logger.error("Exchange name is required")
            return False
        
        # Check if required API credentials exist for live trading
        if not self.sandbox and (not self.api_key or not self.api_secret):
            logger.warning(f"Missing API credentials for {self.name} in live mode")
            return False
            
        return True

@dataclass
class FirebaseConfig:
    """Firebase configuration for state management"""
    project_id: str
    private_key_id: Optional[str] = None
    private_key: Optional[str] = None
    client_email: Optional[str] = None
    
    def validate(self) -> bool:
        """Validate Firebase configuration"""
        if not self.project_id:
            logger.error("Firebase project_id is required")
            return False
            
        # For production, validate service account credentials
        if os.getenv('ENVIRONMENT', 'development') == 'production':
            required_fields = ['private_key_id', 'private_key', 'client_email']
            missing = [field for field in required_fields if not getattr(self, field)]
            if missing:
                logger.error(f"Missing Firebase credentials in production: {missing}")
                return False
                
        return True

class TradingConfig:
    """Main configuration manager for the trading system"""
    
    def __init__(self, config_path: Optional[str] = None):
        """
        Initialize configuration from environment variables and config files
        
        Args:
            config_path: Optional path to configuration file
        """
        self.logger = structlog.get_logger(__name__)
        self.exchanges: Dict[str, ExchangeConfig] = {}
        self.firebase_config: Optional[FirebaseConfig] = None
        
        # Load from environment variables first
        self._load_from_env()
        
        # Load from config file if provided
        if config_path and Path(config_path).exists():
            self._load_from_file(config_path)
        elif config_path:
            self.logger.warning(f"Config file not found: {config_path}")
            
        # Validate configurations
        self._validate_configs()
    
    def _load_from_env(self) -> None:
        """Load configuration from environment variables"""
        # Firebase configuration
        project_id = os.getenv('FIREBASE_PROJECT_ID')
        if project_id:
            self.firebase_config = FirebaseConfig(
                project_id=project_id,
                private_key_id=os.getenv('FIREBASE_PRIVATE_KEY_ID'),
                private_key=os.getenv('FIREBASE_PRIVATE_KEY'),
                client_email=os.getenv('FIREBASE_CLIENT_EMAIL')
            )
            self.logger.info(f"Loaded Firebase config for project: {project_id}")
        
        # Exchange configurations
        exchange_names = os.getenv('ACTIVE_EXCHANGES', 'binance,coinbasepro').split(',')
        
        for exchange in exchange_names:
            exchange = exchange.strip().lower()
            api_key_env = f"{exchange.upper()}_API_KEY"
            api_secret_env = f"{exchange.upper()}_API_SECRET"
            
            self.exchanges[exchange] = ExchangeConfig(
                name=exchange,
                api_key=os.getenv(api_key_env),
                api_secret=os.getenv(api_secret_env),
                sandbox=os.getenv(f"{exchange.upper()}_SANDBOX", "true").lower() == "true"
            )
            self.logger.info(f"Loaded config for exchange: {exchange}")
    
    def _load_from_file(self, config_path: str) -> None:
        """Load configuration from JSON/YAML file (simplified - implement based on format)"""
        # Placeholder for file-based config loading
        self.logger.info(f"Loading configuration from file: {config_path}")
        # Implementation would depend on file format (JSON, YAML, etc.)
    
    def _validate_configs(self) -> None:
        """Validate all configurations"""
        validation_passed = True
        
        # Validate Firebase config
        if self.firebase_config:
            if not self.firebase_config.validate():
                validation_passed = False
        else:
            self.logger.warning("No Firebase configuration found")
        
        # Validate exchange configs
        if not self.exchanges:
            self.logger.error("No exchange configurations found")
            validation_passed = False
        else:
            for