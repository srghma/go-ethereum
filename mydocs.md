what --dev flag is doing

https://github.com/ethereum/go-ethereum/blob/d8ff53dfb8a516f47db37dbc7fd7ad18a1e8a125/cmd/utils/flags.go#L1623-L1667

it only creates a developer acount and makes this account the author of genesis block
AND period 

[entrypoint for no command (start node)](https://github.com/ethereum/go-ethereum/blob/d8ff53dfb8a516f47db37dbc7fd7ad18a1e8a125/cmd/geth/main.go#L203)

`GlobalInt` returns 0 if flag not found (https://github.dev/urfave/cli/tree/v1.22.5 , flag_int.go)

--------

https://github.dev/ethereum/go-ethereum/blob/496f05cf52f2b7f1baaa064eee49aea8ed604ec8/node/node.go#L43

```go
type Node struct {
	http          *httpServer //
	ws            *httpServer //
	httpAuth      *httpServer //
	wsAuth        *httpServer //
	ipc           *ipcServer  // Stores information about the ipc http server

```

https://github.dev/ethereum/go-ethereum/blob/496f05cf52f2b7f1baaa064eee49aea8ed604ec8/node/rpcstack.go#L59-L60

```go
type httpServer struct {

func newHTTPServer(log log.Logger, timeouts rpc.HTTPTimeouts) *httpServer {
```


https://github.dev/ethereum/go-ethereum/blob/496f05cf52f2b7f1baaa064eee49aea8ed604ec8/eth/api_backend.go#L44

```go
func (n *Node) RegisterHandler(name, path string, handler http.Handler) {
...

// graphql/service.go
func (h GraphiQL) ServeHTTP(w http.ResponseWriter, r *http.Request) {

// node/rpcstack.go
func (h *httpServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {

// node/rpcstack.go
// ServeHTTP serves JSON-RPC requests over HTTP, implements http.Handler
func (h *virtualHostHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {

// ServeHTTP serves JSON-RPC requests over HTTP.
func (s *Server) ServeHTTP(w http.ResponseWriter, r *http.Request) {

-------

// node/rpcstack.go

// enableWS turns on JSON-RPC over WebSocket on the server.
func (h *httpServer) enableWS(apis []rpc.API, config wsConfig) error {
		Handler: NewWSHandlerStack(srv.WebsocketHandler(config.Origins), config.jwtSecret), -- WebsocketHandler!!!
   func (s *Server) WebsocketHandler(allowedOrigins []string) http.Handler {

-----------

// node/rpcstack.go

// enableRPC turns on JSON-RPC over HTTP on the server.
func (h *httpServer) enableRPC(apis []rpc.API, config httpConfig) error {
	srv := rpc.NewServer()
		Handler: NewHTTPHandlerStack(srv, config.CorsAllowedOrigins, config.Vhosts, config.jwtSecret), -- NewServer!!!

// Serve serves a vflux request batch
// Note: requests are served by the Handle functions of the registered services. Serve
// may be called concurrently but the Handle functions are called sequentially and
// therefore thread safety is guaranteed.
func (s *Server) Serve(id enode.ID, address string, requests vflux.Requests) vflux.Replies {
--> 			results[i] = service.backend.Handle(id, address, req.Name, req.Params)

--> func (s *Server) Register(b Service, id, desc string) {

-->

type LesServer struct {
	vfluxServer *vfs.Server
	srv.vfluxServer.Register(srv.clientPool, "les", "Ethereum light client service")

	srv.clientPool = vfs.NewClientPool(lesDb, srv.minCapacity, defaultConnectedBias, mclock.System{}, issync)

-->  ???

--------

// API describes the set of methods offered over the RPC interface
type API struct {
	Namespace     string      // namespace under which the rpc methods of Service are exposed
	Version       string      // api version for DApp's
	Service       interface{} // receiver instance which holds the methods
	Public        bool        // indication if the methods must be considered safe for public use
	Authenticated bool        // whether the api should only be available behind authentication.
}


const (
	PendingBlockNumber  = BlockNumber(-2)
	LatestBlockNumber   = BlockNumber(-1)
	EarliestBlockNumber = BlockNumber(0)
)

err := server.RegisterName("eth" OR "shh" OR ..., service)

srv.RegisterName(api.Namespace, api.Service)

// node/rpcstack.go
func RegisterApis(apis []rpc.API, modules []string, srv *rpc.Server, exposeAll bool) error {

// cmd/clef/main.go
		err := node.RegisterApis(rpcAPI, []string{"account"}, srv, false)

---------

func (h *httpServer) enableRPC(apis []rpc.API, config httpConfig) error {
	if err := RegisterApis(apis, config.Modules, srv, false); err != nil {


func (h *httpServer) enableWS(apis []rpc.API, config wsConfig) error {
	if err := RegisterApis(apis, config.Modules, srv, false); err != nil {

// httpConfig is the JSON-RPC/HTTP configuration.
type httpConfig struct {
	Modules            []string
	CorsAllowedOrigins []string
	Vhosts             []string
	prefix             string // path prefix on which to mount http handler
	jwtSecret          []byte // optional JWT secret
}

// wsConfig is the JSON-RPC/Websocket configuration
type wsConfig struct {
	Origins   []string
	Modules   []string
	prefix    string // path prefix on which to mount ws handler
	jwtSecret []byte // optional JWT secret
}

// node/node.go
// for enableWS 
--> func (n *Node) GetAPIs() (unauthenticated, all []rpc.API) {
--> func (n *Node) RegisterAPIs(apis []rpc.API) {

-----------

// apis returns the collection of built-in RPC APIs.
func (n *Node) apis() []rpc.API {
	return []rpc.API{
		{
			Namespace: "admin",
			Version:   "1.0",
			Service:   &privateAdminAPI{n},
		}, {
			Namespace: "admin",
			Version:   "1.0",
			Service:   &publicAdminAPI{n},
			Public:    true,
		}, {
			Namespace: "debug",
			Version:   "1.0",
			Service:   debug.Handler,
		}, {
			Namespace: "web3",
			Version:   "1.0",
			Service:   &publicWeb3API{n},
			Public:    true,
		},
	}
}


	rpcAPI := []rpc.API{
		{
			Namespace: "account",
			Public:    true,
			Service:   api,
			Version:   "1.0"},
	}


func (c *Clique) APIs(chain consensus.ChainHeaderReader) []rpc.API {
	return []rpc.API{{
		Namespace: "clique",
		Version:   "1.0",
		Service:   &API{chain: chain, clique: c},
		Public:    false,
	}}
}


func (ethash *Ethash) APIs(chain consensus.ChainHeaderReader) []rpc.API {
	// In order to ensure backward compatibility, we exposes ethash RPC APIs
	// to both eth and ethash namespaces.
	return []rpc.API{
		{
			Namespace: "eth",
			Version:   "1.0",
			Service:   &API{ethash},
			Public:    true,
		},
		{
			Namespace: "ethash",
			Version:   "1.0",
			Service:   &API{ethash},
			Public:    true,
		},
	}
}


	// Append all the local APIs and return
	return append(apis, []rpc.API{
		{
			Namespace: "eth",
			Version:   "1.0",
			Service:   NewPublicEthereumAPI(s),
			Public:    true,
		}, {
			Namespace: "eth",
			Version:   "1.0",
			Service:   NewPublicMinerAPI(s),
			Public:    true,
		}, {
			Namespace: "eth",
			Version:   "1.0",
			Service:   downloader.NewPublicDownloaderAPI(s.handler.downloader, s.eventMux),
			Public:    true,
		}, {
			Namespace: "miner",
			Version:   "1.0",
			Service:   NewPrivateMinerAPI(s),
			Public:    false,
		}, {
			Namespace: "eth",
			Version:   "1.0",
			Service:   filters.NewPublicFilterAPI(s.APIBackend, false, 5*time.Minute),
			Public:    true,
		}, {
			Namespace: "admin",
			Version:   "1.0",
			Service:   NewPrivateAdminAPI(s),
		}, {
			Namespace: "debug",
			Version:   "1.0",
			Service:   NewPublicDebugAPI(s),
			Public:    true,
		}, {
			Namespace: "debug",
			Version:   "1.0",
			Service:   NewPrivateDebugAPI(s),
		}, {
			Namespace: "net",
			Version:   "1.0",
			Service:   s.netRPCService,
			Public:    true,
		},
	}...)


// internal/ethapi/api.go
// les/api_backend.go
```