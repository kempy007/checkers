# Attempt 3 at checkers

## Verbatim steps

### ignite-cli
```
curl https://get.ignite.com/cli@v0.22.1 | bash
where ignite
sudo mv /usr/local/bin/ignite /usr/local/bin/ignite.v0.24
sudo mv ignite /usr/local/bin
cd /Users/mkemp/repos/cosmosProjects
```
### step1 - create repo called checkers in github
```
ignite scaffold chain github.com/kempy007/checkers
cd checkers
```
### step2
```
git checkout -b stored-object
mkdir x/checkers/rules
curl https://raw.githubusercontent.com/batkinson/checkers-go/a09daeb1548dd4cc0145d87c8da3ed2ea33a62e3/checkers/checkers.go | sed 's/package checkers/package rules/' > x/checkers/rules/checkers.go
git add . && git commit -m "step2"
```
### step3
```
ignite scaffold single systemInfo nextId:uint --module checkers --no-message
git add . && git commit -m "step3"
```
### step4
```
ignite scaffold map storedGame board turn black red --index index --module checkers --no-message
git add . && git commit -m "step4"
```
### step5
<img src="https://raw.githubusercontent.com/zeke/atom-icon/master/old-icon/2.png" width="25" /> ./proto/checkers/genesis.proto

```javascript 
L15-  SystemInfo systemInfo = 2;
L15+  SystemInfo systemInfo = 2 [(gogoproto.nullable) = false];
```

```
git add . && git commit -m "step5"
```

### step6
```
ignite generate proto-go
git add . && git commit -m "step6"
```
### step7
<img src="https://user-images.githubusercontent.com/727262/40395108-6bcc327a-5e1e-11e8-9f76-3917983b8563.png" width="25" /> ./x/checkers/types/genesis.go

```javascript 
L13-		SystemInfo:     nil,
L13+    SystemInfo:     SystemInfo{
L14+      NextId: uint64(DefaultIndex),
L15+    },
```

```
git add . && git commit -m "step7"
```

### step8 - referenced fixes in https://github.com/cosmos/b9-checkers-academy-draft/commit/58f4adc
<img src="https://user-images.githubusercontent.com/727262/40395108-6bcc327a-5e1e-11e8-9f76-3917983b8563.png" width="25" /> ./x/checkers/client/cli/query_system_info_test.go

```javascript
L24-  systemInfo := &types.SystemInfo{}
L24+  systemInfo := types.SystemInfo{}
L30- 	return network.New(t, cfg), *state.SystemInfo
L30+ 	return network.New(t, cfg), state.SystemInfo
```

<img src="https://user-images.githubusercontent.com/727262/40395108-6bcc327a-5e1e-11e8-9f76-3917983b8563.png" width="25" />./x/checkers/genesis.go

```javascript 
L13-  if genState.SystemInfo != nil {
L14-    k.SetSystemInfo(ctx, *genState.SystemInfo)
L15-  }
L13+  k.SetSystemInfo(ctx, genState.SystemInfo)

L32-  genesis.SystemInfo = &systemInfo
L30+  genesis.SystemInfo = systemInfo
```

<img src="https://user-images.githubusercontent.com/727262/40395108-6bcc327a-5e1e-11e8-9f76-3917983b8563.png" width="25" />./x/checkers/genesis_test.go

```javascript 
L17-  SystemInfo: &types.SystemInfo{
L17+  SystemInfo: types.SystemInfo{
```

<img src="https://user-images.githubusercontent.com/727262/40395108-6bcc327a-5e1e-11e8-9f76-3917983b8563.png" width="25" />./x/checkers/types/genesis_test.go

```javascript 
L25-  SystemInfo: &types.SystemInfo{
L25+  SystemInfo: types.SystemInfo{
L67+  func TestDefaultGenesisState_ExpectedInitialNextId(t *testing.T) {
L68+  	require.EqualValues(t,
L69+  		&types.GenesisState{
L70+  			StoredGameList: []types.StoredGame{},
L71+  			SystemInfo:     types.SystemInfo{uint64(1)},
L72+  		},
L73+  		types.DefaultGenesis())
L74+  }
```
```
git add . && git commit -m "step8"
```
### step9 - adding new file

<img src="https://user-images.githubusercontent.com/727262/40395108-6bcc327a-5e1e-11e8-9f76-3917983b8563.png" width="25" />./x/checkers/types/full_game.go **add new file**

```javascript 
package types

import (
	"errors"
	"fmt"

	"github.com/YOURUSER/checkers/x/checkers/rules"
	sdk "github.com/cosmos/cosmos-sdk/types"
	sdkerrors "github.com/cosmos/cosmos-sdk/types/errors"
)

func (storedGame StoredGame) GetBlackAddress() (black sdk.AccAddress, err error) {
	black, errBlack := sdk.AccAddressFromBech32(storedGame.Black)
	return black, sdkerrors.Wrapf(errBlack, ErrInvalidBlack.Error(), storedGame.Black)
}

func (storedGame StoredGame) GetRedAddress() (red sdk.AccAddress, err error) {
	red, errRed := sdk.AccAddressFromBech32(storedGame.Red)
	return red, sdkerrors.Wrapf(errRed, ErrInvalidRed.Error(), storedGame.Red)
}

func (storedGame StoredGame) ParseGame() (game *rules.Game, err error) {
	board, errBoard := rules.Parse(storedGame.Board)
	if errBoard != nil {
		return nil, sdkerrors.Wrapf(errBoard, ErrGameNotParseable.Error())
	}
	board.Turn = rules.StringPieces[storedGame.Turn].Player
	if board.Turn.Color == "" {
		return nil, sdkerrors.Wrapf(errors.New(fmt.Sprintf("Turn: %s", storedGame.Turn)), ErrGameNotParseable.Error())
	}
	return board, nil
}

func (storedGame StoredGame) Validate() (err error) {
	_, err = storedGame.GetBlackAddress()
	if err != nil {
		return err
	}
	_, err = storedGame.GetRedAddress()
	if err != nil {
		return err
	}
	_, err = storedGame.ParseGame()
	return err
}
```

<img src="https://user-images.githubusercontent.com/727262/40395108-6bcc327a-5e1e-11e8-9f76-3917983b8563.png" width="25" />./x/checkers/types/errors.go

```javascript 
L11-  	ErrSample = sdkerrors.Register(ModuleName, 1100, "sample error")
L11+    ErrInvalidBlack     = sdkerrors.Register(ModuleName, 1100, "black address is invalid: %s")
L12+    ErrInvalidRed       = sdkerrors.Register(ModuleName, 1101, "red address is invalid: %s")
L13+    ErrGameNotParseable = sdkerrors.Register(ModuleName, 1102, "game cannot be parsed")
```
```
git add . && git commit -m "step9"
```
### step9a
<img src="https://user-images.githubusercontent.com/727262/40395108-6bcc327a-5e1e-11e8-9f76-3917983b8563.png" width="25" />./x/checkers/types/full_game.go

```javascript 
L7-  "github.com/YOURUSER/checkers/x/checkers/rules"
L7+  "github.com/kempy007/checkers/x/checkers/rules"
```
```
git add . && git commit -m "step9a fix username"
```
### step10
```
go test github.com/kempy007/checkers/x/checkers/keeper
go test github.com/kempy007/checkers/x/checkers/types
cd x/checkers/types/ && go test
```
<img src="https://user-images.githubusercontent.com/727262/40395108-6bcc327a-5e1e-11e8-9f76-3917983b8563.png" width="25" />./x/checkers/types/full_game_test.go **add new file**

```javascript 
package types_test

import (
	"strings"
	"testing"

	"github.com/USERNAME/checkers/x/checkers/rules"
	"github.com/USERNAME/checkers/x/checkers/types"
	sdk "github.com/cosmos/cosmos-sdk/types"
	"github.com/stretchr/testify/require"
)

const (
	alice = "cosmos1jmjfq0tplp9tmx4v9uemw72y4d2wa5nr3xn9d3"
	bob   = "cosmos1xyxs3skf3f4jfqeuv89yyaqvjc6lffavxqhc8g"
)

func GetStoredGame1() types.StoredGame {
	return types.StoredGame{
		Black: alice,
		Red:   bob,
		Index: "1",
		Board: rules.New().String(),
		Turn:  "b",
	}
}

func TestCanGetAddressBlack(t *testing.T) {
	aliceAddress, err1 := sdk.AccAddressFromBech32(alice)
	black, err2 := GetStoredGame1().GetBlackAddress()
	require.Equal(t, aliceAddress, black)
	require.Nil(t, err2)
	require.Nil(t, err1)
}

func TestGetAddressWrongBlack(t *testing.T) {
	storedGame := GetStoredGame1()
	storedGame.Black = "cosmos1jmjfq0tplp9tmx4v9uemw72y4d2wa5nr3xn9d4" // Bad last digit
	black, err := storedGame.GetBlackAddress()
	require.Nil(t, black)
	require.EqualError(t,
		err,
		"black address is invalid: cosmos1jmjfq0tplp9tmx4v9uemw72y4d2wa5nr3xn9d4: decoding bech32 failed: invalid checksum (expected 3xn9d3 got 3xn9d4)")
	require.EqualError(t, storedGame.Validate(), err.Error())
}

func TestCanGetAddressRed(t *testing.T) {
	bobAddress, err1 := sdk.AccAddressFromBech32(bob)
	red, err2 := GetStoredGame1().GetRedAddress()
	require.Equal(t, bobAddress, red)
	require.Nil(t, err1)
	require.Nil(t, err2)
}

func TestGetAddressWrongRed(t *testing.T) {
	storedGame := GetStoredGame1()
	storedGame.Red = "cosmos1xyxs3skf3f4jfqeuv89yyaqvjc6lffavxqhc8h" // Bad last digit
	red, err := storedGame.GetRedAddress()
	require.Nil(t, red)
	require.EqualError(t,
		err,
		"red address is invalid: cosmos1xyxs3skf3f4jfqeuv89yyaqvjc6lffavxqhc8h: decoding bech32 failed: invalid checksum (expected xqhc8g got xqhc8h)")
	require.EqualError(t, storedGame.Validate(), err.Error())
}

func TestParseGameCorrect(t *testing.T) {
	game, err := GetStoredGame1().ParseGame()
	require.EqualValues(t, rules.New().Pieces, game.Pieces)
	require.Nil(t, err)
}

func TestParseGameCanIfChangedOk(t *testing.T) {
	storedGame := GetStoredGame1()
	storedGame.Board = strings.Replace(storedGame.Board, "b", "r", 1)
	game, err := storedGame.ParseGame()
	require.NotEqualValues(t, rules.New().Pieces, game)
	require.Nil(t, err)
}

func TestParseGameWrongPieceColor(t *testing.T) {
	storedGame := GetStoredGame1()
	storedGame.Board = strings.Replace(storedGame.Board, "b", "w", 1)
	game, err := storedGame.ParseGame()
	require.Nil(t, game)
	require.EqualError(t, err, "game cannot be parsed: invalid board, invalid piece at 1, 0")
	require.EqualError(t, storedGame.Validate(), err.Error())
}

func TestParseGameWrongTurnColor(t *testing.T) {
	storedGame := GetStoredGame1()
	storedGame.Turn = "w"
	game, err := storedGame.ParseGame()
	require.Nil(t, game)
	require.EqualError(t, err, "game cannot be parsed: Turn: w")
	require.EqualError(t, storedGame.Validate(), err.Error())
}

func TestGameValidateOk(t *testing.T) {
	storedGame := GetStoredGame1()
	require.NoError(t, storedGame.Validate())
}
```
```
go test
```

<img src="https://user-images.githubusercontent.com/727262/40395108-6bcc327a-5e1e-11e8-9f76-3917983b8563.png" width="25" />./x/checkers/types/full_game_test.go

```javascript 
L8-  	"github.com/USERNAME/checkers/x/checkers/rules"
L9-  	"github.com/USERNAME/checkers/x/checkers/types"
L8+  	"github.com/kempy007/checkers/x/checkers/rules"
L9+ 	"github.com/kempy007/checkers/x/checkers/types"
```

```
go test

checkersd query checkers --help
checkersd query checkers show-system-info
checkersd query checkers show-system-info --output json
checkersd query checkers list-stored-game
git add . && git commit -m "step10"
git push --set-upstream origin stored-object
```