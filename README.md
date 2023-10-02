# Mainnet forking workshop

1. Login into your host (get the details from workshop organizers):
``` bash
ssh workshop@...
```
2. Download `neutrond`:
``` bash
wget https://github.com/neutron-org/neutron/releases/download/v1.0.4/neutrond-linux-amd64 -O ~/neutrond
chmod +x ~/neutrond
```
3. Add the key for testing:
``` bash
MNEMONIC="leopard exclude more together bottom face flight elder trash mushroom hidden win demand fog bubble mosquito capital list dress dwarf erosion puzzle lobster clap"
echo "$MNEMONIC" | neutrond keys add k --recover --keyring-backend=test
```
4. Clone `rehearsal` repo:
``` bash
git clone https://github.com/hadronlabs-org/rehearsal.git
cd rehearsal
```
5. Build the mainnet fork image:
``` bash
make build-mainnet-fork-image
```
6. Download the snapshot:
``` bash
mkdir snapshot
cd snapshot
wget wget http://91.201.113.185:3000/snapshot.json
cd ..
```
7. Run the mainnet fork (will take some time):
``` bash
make start-mainnet-fork
```
8. Check the balance of test account:
``` bash
./neutrond q bank balances neutron1kyn3jx88wvnm3mhnwpuue29alhsatwzrpkwhu6
```
9. Check the voting power of test account (should be 0):
``` bash
./neutrond query wasm contract-state smart neutron1suhgf5svhu4usrurvxzlgn54ksxmn8gljarjtxqnapv8kjnp4nrstdxvff '{
  "voting_power_at_height": {
    "address": "neutron1kyn3jx88wvnm3mhnwpuue29alhsatwzrpkwhu6"
  }
}' -o json | jq
```
10. Bond some NTRN to get voting power:
``` bash
./neutrond tx wasm execute neutron1qeyjez6a9dwlghf9d6cy44fxmsajztw257586akk6xn6k88x0gus5djz4e '{
  "bond": {}
}' --amount 1000000untrn --from k --keyring-backend=test --chain-id neutron-1 
```
11. Check the voting power of test account (should be 1000000):
``` bash
./neutrond query wasm contract-state smart neutron1suhgf5svhu4usrurvxzlgn54ksxmn8gljarjtxqnapv8kjnp4nrstdxvff '{
  "voting_power_at_height": {
    "address": "neutron1kyn3jx88wvnm3mhnwpuue29alhsatwzrpkwhu6"
  }
}' -o json | jq
```
12. Create a proposal:
``` bash
./neutrond tx wasm execute neutron1hulx7cgvpfcvg83wk5h96sedqgn72n026w6nl47uht554xhvj9nsgs8v0z '{
  "propose": {
    "msg": {
      "propose": {
        "description": "description",
        "msgs": [
          {
            "wasm": {
              "execute": {
                "contract_addr": "neutron1f6jlx7d9y408tlzue7r2qcf79plp549n30yzqjajjud8vm7m4vdspg933s",
                "funds": [],
                "msg": "ewogICJhZGRfdm90aW5nX3ZhdWx0IjogewogICAgICAibmV3X3ZvdGluZ192YXVsdF9jb250cmFjdCI6ICJuZXV0cm9uMTN2M2YybnBhdHR6MGR3Mmo1ZjdkbHdzZDh2MmxtY2d5dXYzNTh4OHB2M2plYzRkNjBocnM0eDRkcDciCiAgfQp9"
              }
            }
          },
          {
            "wasm": {
              "execute": {
                "contract_addr": "neutron1suhgf5svhu4usrurvxzlgn54ksxmn8gljarjtxqnapv8kjnp4nrstdxvff",
                "funds": [],
                "msg": "ewogICAgInVwZGF0ZV9zdWJfZGFvcyI6IHsKICAgICAgICAidG9fYWRkIjogWwogICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAiYWRkciI6ICJuZXV0cm9uMXpqZHYzdTZzdmxhemx5ZG1qZTJxY3A0NHlxa3QwMDU5Y2h6OGdteWw1eXJrbG1ndjZmenE5Y2hlbHUiCiAgICAgICAgICAgIH0KICAgICAgICBdLAogICAgICAgICJ0b19yZW1vdmUiOiBbXQogICAgfQp9"
              }
            }
          },
          {
            "bank": {
              "send": {
                "amount": [
                  {
                    "amount": "16000000000000",
                    "denom": "untrn"
                  }
                ],
                "to_address": "neutron1zjdv3u6svlazlydmje2qcp44yqkt0059chz8gmyl5yrklmgv6fzq9chelu"
              }
            }
          },
          {
            "bank": {
              "send": {
                "amount": [
                  {
                    "amount": "4000000000000",
                    "denom": "untrn"
                  }
                ],
                "to_address": "neutron18e8vga4gqaerheumv05xausjctf8qj8suw9l2swm3c5f7z864yysx73nw2"
              }
            }
          }
        ],
        "title": "Launch the Neutron Grants Program (Resubmission of Prop N-10)"
      }
    }
  }
}' --gas 10000000 --amount 1000000000untrn --from k --keyring-backend=test --chain-id neutron-1 
```
13. Query the proposal:
``` bash
./neutrond query wasm contract-state smart neutron1436kxs0w2es6xlqpp9rd35e3d0cjnw4sv8j3a7483sgks29jqwgshlt6zh '{
  "proposal": {
    "proposal_id": 15
  }
}'  -o json | jq
```
14. Vote for the proposal:
``` bash
./neutrond tx wasm execute neutron1436kxs0w2es6xlqpp9rd35e3d0cjnw4sv8j3a7483sgks29jqwgshlt6zh '{
  "vote": {
    "proposal_id": 15,
    "vote": "yes"
  }
}' --from k --keyring-backend=test --chain-id neutron-1 --gas 10000000 
```
15. Check the DAO SubDAOs list (should have 1 item):
``` bash
./neutrond query wasm contract-state smart neutron1suhgf5svhu4usrurvxzlgn54ksxmn8gljarjtxqnapv8kjnp4nrstdxvff '{
  "list_sub_daos": {}
}' -o json | jq
```
16. Execute the proposal:
``` bash
./neutrond tx wasm execute neutron1436kxs0w2es6xlqpp9rd35e3d0cjnw4sv8j3a7483sgks29jqwgshlt6zh '{
    "execute": {
        "proposal_id": 15
    }
}' --from k --keyring-backend=test --chain-id neutron-1 --gas 10000000 
```
17. Check the DAO SubDAOs list (should have 2 items):
``` bash
./neutrond query wasm contract-state smart neutron1suhgf5svhu4usrurvxzlgn54ksxmn8gljarjtxqnapv8kjnp4nrstdxvff '{
  "list_sub_daos": {}
}' -o json | jq
```
