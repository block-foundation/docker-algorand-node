<div>
    <img align="right" src="https://raw.githubusercontent.com/block-foundation/brand/master/logo/logo_gray.png" width="96" alt="Block Foundation Logo">
    <h1 align="left">Block Foundation Docker Images</h1>
    <h3 align="left">Algorand Node</h3>
</div>
<br>

---


Run Algorand in a Docker container

### Build

``` sh
docker build -t algorand .
```

### Running

``` sh
docker run -d -p 8080:8080 --name algorand bjweaver/algorand-node
```

> To run on testnet:

``` sh
docker run -d -p 8080:8080 --name algorand bjweaver/algorand-node:testnet
```

### Status

> To obtain node status:

``` sh
docker exec algorand /algorand/node/goal node status -d /algorand/node/data
```

## API Connection

API will listen on `localhost:8080`

``` sh
curl 127.0.0.1:8080/swagger.json
```

## Disclaimer

**THIS SOFTWARE IS PROVIDED AS IS WITHOUT WARRANTY OF ANY KIND, EITHER EXPRESS OR IMPLIED, INCLUDING ANY IMPLIED WARRANTIES OF FITNESS FOR A PARTICULAR PURPOSE, MERCHANTABILITY, OR NON-INFRINGEMENT.**
