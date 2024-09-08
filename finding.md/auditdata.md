### [S-#] Storing the `password onchain` mamkes it visible to anyone, & no longer private. 

**Description:** All data staored onchain is visible to anyone, and can be read directly from the blockchain. The `passwordStore::s_password` variable is intented to be a private variable and only accessed through the `passwordStore::getPassword` function, which is intented to be only called by the owner of the contract.

We show one such method of reading data offchain below. 

**Impact:**  Anyone can read the private password, severely breaking the functionality of the protocol.

**Proof of Concept:** (Proof of code)

1. Create a locally running chain
```bash
make anvil
```
2. Deploy the contract to the chain 
```make deploy```

3. Run the storage tool
```cast storage <Address of the contract> 1 --rpc-url http://127.0.0.1:8545  
You'll get an output that looks like this 

`0x506c6179626f690000000000000000000000000000000000000000000000000e`
You can then parse a hex to a string with 

```cast parse-bytes32-string 0x506c6179626f690000000000000000000000000000000000000000000000000e````
you would get an output like: 
Playboi


**Recommended Mitigation:** Due to this, the overall architecture of the contract should be rethought. One could encrypt the passwprd offchain, and then store the encrypted password onchain.This would require the user to remember another password offchain to decrypt the password. However, you'd also likely want to remove the view function as you wouldnt want the user to accidentally send a tx with the password to decrypt your password. 

### [S-#] `PasswordStore::setPassword` has no access controls, meaning a non-owner could change the password. 

**Description:** The `PasswordStore::setPassword` function is set be an `external` function, however the natspec of the function & overall purpose of the smart contract is that this function allows only the owner to set a new password.

```javascript
 function setPassword(string memory newPassword) external {
 @>   //audit-high
        s_password = newPassword;
        emit SetNewPassword();
    }
 ```   

**Impact:** Anyone can set/change the password of the contract, severely breaking the contract intended functionality.

**Proof of Concept:** Add the following to the `PasswordStore.t.sol` file.

<details>

``` javascript
 function test_anyone_can_set_password(address randomAddress) public{
        vm.assume(randomAddress != owner);
        vm.prank(randomAddress);
        string memory expectedPassword = "myNewPassword";
        passwordStore.setPassword(expectedPassword);

        vm.prank(owner);
        string memory actualPassword = passwordStore.getPassword();
        assertEq(actualPassword, expectedPassword);
    }
``` 
</details>


**Recommended Mitigation:** Add an access control conditional to the `setPassword`function.

``` javascript
if(msg.sender != s_owner){
    revert PasswordStore_Notowner();
}
```