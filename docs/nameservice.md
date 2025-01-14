!!! info
    
    This is a sample SDK help document targeting developers.


# x/nameservice

## Abstract

`x/nameservice` is an implementation of a Cosmos SDK module that allows users to buy unused names, set a name value, or sell/trade their name. The owner of a given name is the current highest bidder. When someone buys a name, they are required to pay the previous owner of the name a price higher than the price the previous owner paid for it.

If a name does not have a previous owner yet, the buyer must burn (send to an unrecoverable address) a `MinPrice` amount.

## Concepts

The `x/nameservice` module uses one store to map `name` to its respective `whois` struct. Each name will have three pieces of data associated with it:

-   `Value` - The value that a name resolves to. This string can be used to require a specific format, such as an IP address, DNS Zone file, or blockchain address.

-   `Owner` - The address of the current owner of the name.

-   `Price` - The price you will need to pay in order to buy the name.

```json linenums="1"
type Whois struct {
    Value string         `json:"value" yaml:"value"`
    Owner sdk.AccAddress `json:"owner" yaml:"owner"`
    Price sdk.Coins      `json:"price" yaml:"price"`
}
```

If a name does not already have an owner, it should be initialized with a `MinPrice`. `MinPrice` is defined as an initial starting price for a name that was never previously owned.

```py linenums="1"

    // MinNamePrice is Initial Starting Price for a name that was never previously owned
    var MinNamePrice = sdk.Coins{sdk.NewInt64Coin("nametoken", 1)}

    // NewWhois returns a new Whois with the minprice as the price
    func NewWhois() Whois {
        return Whois{
            Price: MinNamePrice,
        }
```

## State

### Owner

The owner is the current possessor of the name. It is stored when the name is initially set or when the name is bought by the highest bidder. This uses the `sdk.AccAddress` type which represents an account controlled by public key cryptograhy.

### Price

The price is the current value of the name. It is stored when the initial price is set as `minprice` or when the name is bought by the highest bidder. This uses the `sdk.Coins` type.

## Messages

### MsgSetName

Name owners set a value for a given name with the `MsgSetName` message. `MsgSetName` has three attributes needed to set the value for a name:

-   `Name` - The name trying to be set.

-   `Value` - What the name resolves to.

-   `Owner` - The account of the name owner.

```py linenums="1"
// MsgSetName defines a SetName message
type MsgSetName struct {
    Name  string         `json:"name"`
    Value string         `json:"value"`
    Owner sdk.AccAddress `json:"owner"`
}
```

### MsgBuyName

Accounts can buy a name and become its owner with the `MsgBuyName` message. `MsgBuyName` has three attributes needed to buy a name:

-   `Name` - The name being bid on.

-   `Bid` - The buyers offer to purchase the name.

-   `Buyer` - The account of the potential buyer.

```py linenums="1"
// MsgBuyName defines the BuyName message
type MsgBuyName struct {
    Name  string         `json:"name"`
    Bid   sdk.Coins      `json:"bid"`
    Buyer sdk.AccAddress `json:"buyer"`
}
```

### MsgDeleteName

Names can be deleted with the `MsgDeleteName` message. `MsgDeleteName` message requires two attributes to delete a name:

-   `Name` - The name trying to be set.

-   `Owner` - The account of the name owner.

```py linenums="1"
// This Msg deletes a name.
type MsgDeleteName struct {
    Name  string         `json:"name" yaml:"name"`
    Owner sdk.AccAddress `json:"owner" yaml:"owner"`
}
```

## Parameters

The `x/nameservice` module contains the following parameters:

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Key</th>
<th>Type</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p>Name</p></td>
<td><p>String</p></td>
</tr>
<tr class="even">
<td><p>Value</p></td>
<td><p>String</p></td>
</tr>
<tr class="odd">
<td><p>Owner</p></td>
<td><p>sdk.AccAddress</p></td>
</tr>
<tr class="even">
<td><p>Bid</p></td>
<td><p>sdk.Coins</p></td>
</tr>
<tr class="odd">
<td><p>Buyer</p></td>
<td><p>sdk.AccAddress</p></td>
</tr>
</tbody>
</table>
