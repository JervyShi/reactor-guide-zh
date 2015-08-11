# Environment and Dispatchers
The functional building blocks in place, we can start playing asynchronously with them. First stop is bringing us to the Dispatcher section.

Before we can start any Dispatcher, we want to make sure we create them efficiently. Usually Dispatchers are expensive to create as they will pre-allocate a segment of memory to hold the dispatched signals, the famous runtime vs startup trade-off introduced in the preface. A specific shared context named **Environment** has been introduced to manage these various dispatchers, thus avoiding inapproriate creations.

