
As AI has become more prevalent in our society, many are trying to figure out the best way to adopt this new technology into their personal and professional work flows. Enter the AI assistant - like a regular chatbot, but with internet access. These assistants are designed to integrate directly into development environments, offering real-time assistance.

Throughout this quarter, I have experimented with the built-in JetBrains AI plugin that comes with PyCharm to see how it could be of assistance in my coding endeavors.

---
## Models

There are several models that can be used within the plugin, from Claude to Gemini to GPT. Each AI model provides a different experience for different tasks. After a little research (a.k.a., Google AI overview), the Claude Sonnet model appeared to be the best choice for programming, and my experience confirmed this. I used the Claude Sonnet 3.5 model, and it was the only model able to answer my questions within a reasonable time.

-------
## Prompting
The AI plugin works by providing prompts such as “explain this code”.  It works just like ChatGPT, and will answer just about anything.   

But how useful is this? I set out to see if this AI assistant would be able to help me with a refactor. The task was to come up with a class structure that made sense for our iqe-rhsm-subscriptions-plugin repository.   

Here are some of the prompts, I used.  

I started off asking for a refactor of a single function, as I was curious what type of changes it would make.

_Prompt_: “Refactor this code” .   
```
def get_today_tally_report_for_all_metric(self, product_id, granularity="Daily", **kwargs):
        """
        Get tally report for all metrics in a dict
           :param str product_id: The product id we wish to query
           :param str granularity: The level of granularity to return.
           return: tally report for all metrics in a dictionary
        """
        tally_usage = {
            "sockets": self.get_today_tally_report(
                product_id, granularity, metric_id="Sockets", **kwargs
            )["value"],
            "cores": self.get_today_tally_report(
                product_id, granularity, metric_id="Cores", **kwargs
            )["value"],
            "physical_sockets": self.get_today_tally_report(
                product_id, granularity, metric_id="Sockets", category="physical", **kwargs
            )["value"],
            "hypervisor_sockets": self.get_today_tally_report(
                product_id, granularity, metric_id="Sockets", category="hypervisor", **kwargs
            )["value"],
            "virtual_sockets": self.get_today_tally_report(
                product_id, granularity, metric_id="Sockets", category="virtual", **kwargs
            )["value"],
            "cloud_sockets": self.get_today_tally_report(
                product_id, granularity, metric_id="Sockets", category="cloud", **kwargs
            )["value"],
            "physical_cores": self.get_today_tally_report(
                product_id, granularity, metric_id="Cores", category="physical", **kwargs
            )["value"],
            "virtual_cores": self.get_today_tally_report(
                product_id, granularity, metric_id="Cores", category="virtual", **kwargs
            )["value"],
            "cloud_cores": self.get_today_tally_report(
                product_id, granularity, metric_id="Cores", category="cloud", **kwargs
            )["value"],
            "hypervisor_cores": self.get_today_tally_report(
                product_id, granularity, metric_id="Cores", category="hypervisor", **kwargs
            )["value"],
        }
        return tally_usage
```

_Response_:
```
def get_today_tally_report_for_all_metric(self, product_id, granularity="Daily", **kwargs):
    """
    Get tally report for all metrics in a dict
    
    Args:
        product_id (str): The product id we wish to query
        granularity (str): The level of granularity to return (defaults to 'Daily')
        **kwargs: Additional keyword arguments to pass to get_today_tally_report
    
    Returns:
        dict: Tally report for all metrics containing sockets and cores data for different categories
    """
    metrics = {
        'Sockets': ['physical', 'hypervisor', 'virtual', 'cloud'],
        'Cores': ['physical', 'hypervisor', 'virtual', 'cloud']
    }
    
    tally_usage = {}
    
    # Add total metrics without category
    for metric_id in metrics:
        metric_key = metric_id.lower()
        tally_usage[metric_key] = self.get_today_tally_report(
            product_id, 
            granularity, 
            metric_id=metric_id, 
            **kwargs
        )['value']
    
    # Add category-specific metrics
    for metric_id, categories in metrics.items():
        metric_key = metric_id.lower()
        for category in categories:
            key = f"{category}_{metric_key}"
            tally_usage[key] = self.get_today_tally_report(
                product_id,
                granularity,
                metric_id=metric_id,
                category=category,
                **kwargs
            )['value']
    
    return tally_usage
```

_AI Explanation_:
>Key improvements in this refactored version:
>
>1. **Better Organization**: The metrics and their categories are defined in a single dictionary structure, making it easier to maintain and modify.
  >  
>2. **Reduced Repetition**: Instead of repeating similar calls multiple times, we use loops to generate the metrics.
  >  
>3. **Improved Documentation**: Added detailed docstring with proper Args and Returns sections.
  >  
>4. **More Flexible**: The structure makes it easier to add new metrics or categories by simply updating the dictionary. `metrics`
  >  
>5. **Consistent Naming**: Uses a consistent pattern for generating keys in the result dictionary.
  > To use this function:
```
tally_report = self.get_today_tally_report_for_all_metric('product123')
```
>The output structure remains the same as before, but the code is now more maintainable and less prone to errors. If you need to add new metrics or categories in the future, you only need to update the dictionary at the top of the function. `metrics`  

---
From here, I turned my focus to the top level __ init __ _.py file. 

_Prompt_: " Can you recommend a way to break up this file"  

The AI Assistant took the file as input.

_Response_:
Based on the file structure shown, I recommend breaking up the large `ApplicationRhsmSubscriptions` class into several smaller, focused modules. Here's a suggested organization:

```

rhsm_subscriptions/
├── __init__.py
├── core/
│   ├── __init__.py
│   ├── exceptions.py      # ApplicationRhsmSubscriptionsException
│   └── base.py           # Base class with common attributes
├── api/
│   ├── __init__.py
│   ├── rest_client.py    # REST client related methods
│   ├── ocm.py           # OCM API related methods
│   ├── candlepin.py     # Candlepin API methods
│   └── wiremock.py      # Wiremock related methods
├── services/
│   ├── __init__.py
│   ├── metrics/
│   │   ├── __init__.py
│   │   ├── capacity.py   # Capacity report methods
│   │   └── tally.py     # Tally report methods
│   ├── subscription/
│   │   ├── __init__.py
│   │   ├── manager.py    # Subscription management
│   │   └── sync.py      # Sync related methods
│   └── telemeter/
│       ├── __init__.py
│       └── client.py     # Telemeter related methods
├── mock/
│   ├── __init__.py
│   ├── clusters.py      # Mock cluster creation
│   ├── hosts.py        # Mock host creation
│   └── events.py       # Mock event creation
└── utils/
    ├── __init__.py
    ├── config.py       # Configuration related utilities
    └── validators.py   # Validation helpers
```
> Example of how to split the code:
```

# core/base.py
class ApplicationRhsmSubscriptionsBase:
    plugin_real_name = "rhsm-subscriptions"
    plugin_name = "rhsm_subscriptions"
    plugin_title = "RHSM Subscriptions"
    
    def __init__(self):
        self.tags = []
        self.used_subscriptions = {}

# api/rest_client.py
from ..core.base import ApplicationRhsmSubscriptionsBase

class RestClientMixin(ApplicationRhsmSubscriptionsBase):
    def rest_client(self):
        # REST client implementation
        pass

# services/metrics/capacity.py
from typing import Dict, Any
from ...core.base import ApplicationRhsmSubscriptionsBase

class CapacityService(ApplicationRhsmSubscriptionsBase):
    def get_today_capacity_report_for_all_metric(
        self, product_id: str, granularity: str = "Daily", **kwargs
    ) -> Dict[str, Any]:
        # Implementation
        pass

# services/metrics/tally.py
class TallyService(ApplicationRhsmSubscriptionsBase):
    def get_today_tally_report_for_all_metric(
        self, product_id: str, granularity: str = "Daily", **kwargs
    ) -> Dict[str, Any]:
        # Implementation
        pass

# mock/clusters.py
class ClusterMockService(ApplicationRhsmSubscriptionsBase):
    def create_mock_openshift_cluster(self):
        # Implementation
        pass

# __init__.py
from .core.base import ApplicationRhsmSubscriptionsBase
from .api.rest_client import RestClientMixin
from .services.metrics.capacity import CapacityService
from .services.metrics.tally import TallyService
# ... other imports

class ApplicationRhsmSubscriptions(
    RestClientMixin,
    CapacityService,
    TallyService,
    # ... other mixins
):
    """Main class that combines all functionality through inheritance"""
    pass
```

>Benefits of this organization:
>
>1. **Modularity**: Each module has a single responsibility
>2. **Maintainability**: Easier to find and fix issues
>3. **Testability**: Can test each component independently
>4. **Scalability**: Easier to add new features
>5. **Readability**: Smaller, focused files are easier to understand
>6. **Reusability**: Components can be used independently
>7. **Dependencies**: Clearer dependency management
>
>When implementing this structure:
>
>- Move related methods together
>- Use mixins for shared functionality
>- Create proper interfaces/base classes
>- Maintain backward compatibility
>- Add proper type hints
>- Include comprehensive documentation
>- Add proper test coverage for each module
>
This organization will make the code more maintainable and easier to work with in the long term.

---

I soon started to to ask more descriptive questions in an attempt to sway the AI to provide a code sample that I liked.  

_Prompt_: "I would like to add a proper class structure to this file. Functions should be grouped in the correct class with functions that hit the same services."

_Response_:

```
import json
import logging
import requests
from typing import Dict, Optional, Any, List

log = logging.getLogger(__name__)

class ApplicationRhsmSubscriptionsException(Exception):
    """Base exception for RHSM Subscriptions application"""
    pass

class BaseService:
    """Base class for all services with common functionality"""
    def __init__(self, rest_client=None, rest_v2_client=None):
        self.rest_client = rest_client
        self.rest_v2_client = rest_v2_client

class OCMService(BaseService):
    """OpenShift Cluster Manager service operations"""
    
    def ocm_get_token(self, *args, **kwargs):
        """Get OCM token"""
        # Implementation

    def ocm_list_clusters(self, *args, **kwargs):
        """List OCM clusters"""
        # Implementation

    def extend_stage_cluster_expiration_date(self, *args, **kwargs):
        """Extend cluster expiration date"""
        # Implementation

    def ocm_get_subscription_settings(self, *args, **kwargs):
        """Get OCM subscription settings"""
        # Implementation

    def ocm_change_subscription_settings(self, *args, **kwargs):
        """Change OCM subscription settings"""
        # Implementation

    def ocm_delete_cluster(self, *args, **kwargs):
        """Delete OCM cluster"""
        # Implementation

class TallyService(BaseService):
    """Tally-related operations"""
    
    def get_tally_report(self, *args, **kwargs):
        """Get tally report"""
        # Implementation

    def get_tally_hourly_report_total(self, *args, **kwargs):
        """Get hourly tally report total"""
        # Implementation
    
    def sync_tally(self, *args, **kwargs):
        """Sync tally data"""
        # Implementation

    def sync_tally_hourly(self, *args, **kwargs):
        """Sync hourly tally data"""
        # Implementation

class CapacityService(BaseService):
    """Capacity-related operations"""
    
    def get_capacity_report(self, *args, **kwargs):
        """Get capacity report"""
        # Implementation

    def sync_capacity(self, *args, **kwargs):
        """Sync capacity data"""
        # Implementation

    def sync_capacity_with_subscription_id(self, *args, **kwargs):
        """Sync capacity with subscription ID"""
        # Implementation

class InventoryService(BaseService):
    """Host inventory related operations"""
    
    def get_host_inventory_count(self, *args, **kwargs):
        """Get host inventory count"""
        # Implementation

    def get_all_host_inventory(self, *args, **kwargs):
        """Get all host inventory"""
        # Implementation

    def create_mock_hbi_hosts(self, *args, **kwargs):
        """Create mock HBI hosts"""
        # Implementation

class SubscriptionService(BaseService):
    """Subscription management operations"""
    
    def add_sku(self, *args, **kwargs):
        """Add SKU"""
        # Implementation

    def get_account_skus(self, *args, **kwargs):
        """Get account SKUs"""
        # Implementation

    def get_skus_report(self, *args, **kwargs):
        """Get SKUs report"""
        # Implementation

class ApplicationRhsmSubscriptions:
    """Main application class that orchestrates all services"""
    
    def __init__(self):
        self.plugin_real_name = "rhsm-subscriptions"
        self.plugin_name = "subscriptions"
        self.plugin_title = "Subscriptions"
        self.plugin_package_name = "rhsm-subscriptions"
        
        # Initialize service clients
        self.rest_client = None  # Initialize actual client
        self.rest_v2_client = None  # Initialize actual client
        self.rest_contracts_client = None
        self.rest_contracts_v2_client = None
        
        # Initialize services
        self.ocm_service = OCMService(self.rest_client, self.rest_v2_client)
        self.tally_service = TallyService(self.rest_client, self.rest_v2_client)
        self.capacity_service = CapacityService(self.rest_client, self.rest_v2_client)
        self.inventory_service = InventoryService(self.rest_client, self.rest_v2_client)
        self.subscription_service = SubscriptionService(self.rest_client, self.rest_v2_client)
        
        # Other attributes
        self.tags = []
        self.used_subscriptions = {}

    def __getattr__(self, name):
        """Delegate method calls to appropriate service"""
        for service in [self.ocm_service, self.tally_service, self.capacity_service, 
                       self.inventory_service, self.subscription_service]:
            if hasattr(service, name):
                return getattr(service, name)
        raise AttributeError(f"'{self.__class__.__name__}' object has no attribute '{name}'")
```

_Same prompt, different response_:
```
import json
import requests

class ApiClient:
    def __init__(self, rhsm_service, rhsm_service_cert, rhsm_service_key):
        self.rhsm_service = rhsm_service
        self.rhsm_service_cert = rhsm_service_cert
        self.rhsm_service_key = rhsm_service_key

class OCMService:
    def __init__(self, api_client):
        self.client = api_client
    
    def get_token(self):
        """OCM token related operations"""
        pass

    def list_clusters(self):
        """OCM cluster listing operations"""
        pass
    
    def get_subscription_settings(self):
        """OCM subscription settings operations"""  
        pass

    def change_subscription_settings(self):
        """Change OCM subscription settings"""
        pass

    def delete_cluster(self):
        """Delete OCM cluster operations"""
        pass

class CandlepinService:
    def __init__(self, api_client):
        self.client = api_client
        
    def create_and_wait_for_consumer(self):
        """Candlepin consumer creation operations"""
        pass

    def check_consumer(self):
        """Check Candlepin consumer operations"""
        pass

    def list_systems(self):
        """List Candlepin systems operations"""
        pass

class TelemetryService:
    def __init__(self, api_client):
        self.client = api_client

    def get_current_clusters(self):
        """Get current telemetry clusters"""
        pass

    def get_clusters_system_table(self):
        """Get clusters system table operations"""
        pass

    def get_daily_tally(self):
        """Get daily tally operations"""
        pass

    def get_today_tally_report(self):
        """Get today's tally report operations"""
        pass

class RHSMSubscriptionService:
    def __init__(self, api_client):
        self.client = api_client
        self.ocm = OCMService(api_client)
        self.candlepin = CandlepinService(api_client)
        self.telemetry = TelemetryService(api_client)
    
    def get_capacity_report(self):
        """Get capacity report operations"""
        pass

    def get_tally_report(self):
        """Get tally report operations"""
        pass

    def get_instances_report(self):
        """Get instances report operations"""
        pass

    def sync_swatch(self):
        """Sync swatch operations"""
        pass

    def sync_tally(self):
        """Sync tally operations"""
        pass

class SystemService:
    def __init__(self, rhsm_service, rhsm_service_cert, rhsm_service_key):
        self.rhsm_service = rhsm_service 
        self.rhsm_service_cert = rhsm_service_cert
        self.rhsm_service_key = rhsm_service_key

    def list_systems_for_users(self, username):
        """Get System list for user using rhsm-api
        
        :param str username: username
        """
        url = f"https://{self.rhsm_service}/v1/systems"
        header = {"X-RhsmApi-Username": username}
        response = requests.get(
            url,
            headers=header, 
            cert=(self.rhsm_service_cert, self.rhsm_service_key),
        )
        response.raise_for_status()
        if response.ok:
            res_count = json.loads(response.text)["pagination"]["count"]
            if res_count > 0:
                result = json.loads(response.text)["body"]
                return result
            else:
                raise Exception("No Systems Present.")
```

---
## Take Aways  

#### The Good  

- "Explain this code" is a very good way to have the code summarized to you step by step. It's extremely useful in Python, because of it's often implicit nature
- "Shorten/rewrite this code" is useful in making logic paths more readable  
- Asking the same question will give you different answers, so you can get multiple implementations of the same thing  

This AI assistant does adhere to "standard coding practices". While I wasn't able to refactor, I did find value in just seeing how the AI would structure code. As such, I was able to make an improvement of adding type hinting to our repo, allowing for better pre runtime linting, and ease of knowing what variable types should be passed in to a function.   
[MR1](https://gitlab.cee.redhat.com/insights-qe/iqe-rhsm-subscriptions-plugin/-/merge_requests/1187)  
[MR2](https://gitlab.cee.redhat.com/insights-qe/iqe-rhsm-subscriptions-plugin/-/merge_requests/1178)  

Prompting a refactor on `get_today_capacity_report_for_all_metric()` got me a response that contained type hinting:   
```
from typing import Dict, Any, Optional

class CapacityReporter:
    # Constants for metrics and categories
    METRIC_SOCKETS = "Sockets"
    METRIC_CORES = "Cores"
    CATEGORY_PHYSICAL = "physical"
    CATEGORY_HYPERVISOR = "hypervisor"

    def _get_metric_value(
            self,
            product_id: str,
            granularity: str,
            metric_id: str,
            category: Optional[str] = None,
            **kwargs: Any
    ) -> int:
        """Helper method to get capacity report value for a specific metric"""
        params = {"metric_id": metric_id, **kwargs}
        if category:
            params["category"] = category
        return self.get_today_capacity_report(product_id, granularity, **params)["value"]

    def get_today_capacity_report_for_all_metrics(
            self, product_id: str, granularity: str = "Daily", **kwargs: Any
    ) -> Dict[str, int]:
        """
        Get capacity report for all metrics in a dict
        
        Args:
            product_id: The product id we wish to query
            granularity: The level of granularity to return
            **kwargs: Additional parameters for the capacity report
        
        Returns:
            Dictionary containing capacity values for different metrics
        """
        return {
            "sockets": self._get_metric_value(
                product_id, granularity, self.METRIC_SOCKETS, **kwargs
            ),
            "cores": self._get_metric_value(
                product_id, granularity, self.METRIC_CORES, **kwargs
            ),
            "physical_sockets": self._get_metric_value(
                product_id, granularity, self.METRIC_SOCKETS, self.CATEGORY_PHYSICAL, **kwargs
            ),
            "hypervisor_sockets": self._get_metric_value(
                product_id, granularity, self.METRIC_SOCKETS, self.CATEGORY_HYPERVISOR, **kwargs
            ),
            "physical_cores": self._get_metric_value(
                product_id, granularity, self.METRIC_CORES, self.CATEGORY_PHYSICAL, **kwargs
            ),
            "hypervisor_cores": self._get_metric_value(
                product_id, granularity, self.METRIC_CORES, self.CATEGORY_HYPERVISOR, **kwargs
            ),
        }
```

#### The Bad
- The AI can't process larger files/snippets. Anything more than 1000 lines usually resulted in boiler plate code for me
-  My laptop had performance issues while using the assistant. I actually had 2 total freezes and a handful of slowdowns

#### Closing
I can see myself using this tool for small code snippets and documentation. I think I would need the Pro version and a better performing laptop before I made it a normal part of my workflow.
