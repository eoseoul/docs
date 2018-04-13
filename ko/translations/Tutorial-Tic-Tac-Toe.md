# 틱-택-토

### 목적

아래의 튜토리얼은 사용자가 간단한 플레이어 대 플레이어 게임 컨트랙트를 구축하는 샘플을 만드는 가이드입니다. 틱 택 토 게임을 이용해 이 과정을 보여드리겠습니다. 이 튜토리얼의 최종 결과는 [다음](https://github.com/EOSIO/eos/tree/master/contracts/tic_tac_toe)에서 보실 수 있습니다.


### 가정
이 게임을 위해 3x3 틱 택 토 판을 사용하겠습니다. 플레이어는 두 역할로 나뉩니다: **호스트**, **도전자**. 호스트는 항상 먼저 행동합니다. 각 플레이어 쌍(호스트-도전자)은 최대 동시에 두 게임까지**만** 진행할 수 있습니다. 한 쪽 게임에서는 첫 번째 플레이어가 호스트가 되며, 다른 쪽 게임에서는 두 번째 플레이어가 호스트가 됩니다.


#### 판
보통 틱 택 토 게임에 사용되는 `o`, `x` 표시 대신, 호스트의 행동을 `1`로 표시하고, 도전자의 행동을 `2`로 표시하며, 빈 칸은 `0`으로 표시하겠습니다. 아울러, 판의 상황을 저장하기 위해 1차원 배열을 사용할 것입니다. 즉:

|           | (0,0) | (1,0) | (2,0) |
| :-------: | :---: | :---: | :---: |
| **(0,0)** | -     | o     | x     |
| **(0,1)** | -     | x     | -     |
| **(0,2)** | x     | o     | o     |

x가 호스트일 때, 위의 판은 다음 배열이 됩니다: `[0, 2, 1, 0, 1, 0, 1, 2, 2]`

#### 액션
플레이어는 이 컨트랙트에서 상호작용하기 위해 다음과 같은 액션을 쓸 수 있습니다:
- create: 새 게임을 시작합니다
- restart: 진행중인 게임을 재시작합니다, 호스트나 도전자가 모두 이 액션을 실행할 수 있습니다
- close: 현재 게임을 종료하며, 게임을 저장하기 위해 사용한 저장 공간을 반납합니다. 호스트만이 이 액션을 실행할 수 있습니다
- move: 행동합니다


#### 컨트랙트 계정
가이드에서 이후의 진행을 위해 컨트랙트를 `tic.tac.toe` 계정에 밀어넣을 것입니다. `tic.tac.toe` 계정명을 이미 누군가 사용중이라면, 다른 계정명을 만들고 코드 내의 `tic.tac.toe` 부분을 사용하고자 하는 계정명으로 바꾸시면 되겠습니다. 계정을 아직 만들지 않으셨다면, 일단 먼저 만드세요.

```bash
$ cleos create account ${creator_name} ${contract_account_name} ${contract_pub_owner_key} ${contract_pub_active_key} --permission ${creator_name}@active
# e.g. $ cleos create account inita tic.tac.toe  EOS4toFS3YXEQCkuuw1aqDLrtHim86Gz9u3hBdcBw5KNPZcursVHq EOS7d9A3uLe6As66jzN8j44TXJUqJSK3bFjjEEqR4oTvNAB3iM9SA --permission inita@active
```
지갑이 잠금해제돼 있는지 확인하시고, 생성자의 개인 활성화 키가 지갑에 임포트됐는지도 확인하세요. 안그러면 위의 명령어는 실행되지 못합니다.

### 시작!
세 파일을 만들도록 하겠습니다:
- tic_tac_toe.hpp => 컨트랙트에서 사용하는 구조체들이 정의된 헤더 파일
- tic_tac_toe.cpp => 컨트랙트의 주요 로직
- tic_tac_toe.abi => 플레이어가 컨트랙트와 상호작용하기 위한 인터페이스
주의: 이 예제에서는 컨트랙트 계정명으로 N(tic.tac.toe)를 사용했습니다. 만약 컨트랙트 계정에 다른 이름을 사용하시려면, `tic.tac.toe` 부분을 사용하실 계정 명으로 바꿔주세요.


### 구조체 정의
헤더 파일부터 시작합시다. 일단 컨트랙트의 구조체를 정의합니다. **tic_tac_toe.hpp** 파일을 열어서 아래의 boilerplate 처럼 시작하세요.

```cpp
// Import necessary library
#include <eosiolib/eosio.hpp> // Generic eosio library, i.e. print, type, math, etc

using namespace eosio;
namespace tic_tac_toe {
   static const account_name games_account = N(games);
   static const account_name code_account = N(tic.tac.toe);
    // Your code here
}
```

#### 게임 테이블
이 컨트랙트를 위해 게임 리스트를 저장하는 테이블이 필요합니다. 이렇게 해봅시다:
```cpp
...
namespace tic_tac_toe {
    ...
    typedef eosio::multi_index< games_account, game> games;
}
  
```
- 템플릿 파라미터의 첫 번째는 테이블의 이름입니다
- 템플릿 파라미터의 두 번째는 테이블이 저장할 구조체(다음 섹션에서 정의될)입니다


#### 게임 구조체
게임에 쓰일 구조체를 정의합시다. 이 구조체 정의가 코드 상에서 테이블 정의보다 반드시 앞에 있어야 합니다.
```cpp
...
namespace tic_tac_toe {
   static const uint32_t board_len = 9;
   struct game {
      game() {}
      game(account_name challenger, account_name host):challenger(challenger), host(host), turn(host) {
         // Initialize board
         initialize_board();
      }
      account_name     challenger;
      account_name     host;
      account_name     turn; // = account name of host/ challenger
      account_name     winner = N(none); // = none/ draw/ account name of host/ challenger
      uint8_t          board[9]; //

      // Initialize board with empty cell
      void initialize_board() {
         for (uint8_t i = 0; i < board_len ; i++) {
            board[i] = 0;
         }
      }

      // Reset game
      void reset_game() {
         initialize_board();
         turn = host;
         winner = N(none);
      }

      auto primary_key() const { return challenger; }

      EOSLIB_SERIALIZE( game, (challenger)(host)(turn)(winner)(board) )
   };
}
```
primary_key 메소드는 위의 게임을 위한 테이블 정의에서 필요한 것입니다. 테이블에서 어떤 필드가 검색에 사용되는 키인지 알 수 있게 합니다.


#### 액션 구조체
##### Create
게임을 시작하기 위해서는 호스트 계정명과 도전자 계정명이 필요합니다. EOSLIB_SERIALIZE 매크로에는 액션을 컨트랙트와 nodeos 시스템 간에 전달할 수 있도록 직렬화/역직렬화 메소드가 포함돼 있습니다.

```cpp
...
namespace tic_tac_toe {
   ...
   struct create {
      account_name   challenger;
      account_name   host;

      EOSLIB_SERIALIZE( create, (challenger)(host) )
   };
   ...
}
```
##### Restart
게임을 재시작하기 위해서는 게임을 식별할 수 있도록 호스트 계정명과 도전자 계정명이 필요합니다. 아울러, 누가 게임을 재시작하고자 하는지 지정해야 합니다. 그래야 포함된 서명의 유효성을 검증할 수 있습니다.
```cpp
...
namespace tic_tac_toe {
   ...
   struct restart {
      account_name   challenger;
      account_name   host;
      account_name   by;

      EOSLIB_SERIALIZE( restart, (challenger)(host)(by) )
   };
   ...
}
```
##### Close
게임을 종료하기 위해서는 게임을 식별할 수 있도록 호스트 계정명과 도전자 계정명이 필요합니다.
```cpp
...
namespace tic_tac_toe {
   ...
   struct close {
      account_name   challenger;
      account_name   host;

      EOSLIB_SERIALIZE( close, (challenger)(host) )
   };
   ...
}
```
##### Move
행동을 위해서는 게임을 식별할 수 있도록 호스트 계정명과 도전자 계정명이 필요합니다. 아울러, 누가 행동을 했고 어떤 행동을 했는지 지정해 주어야 합니다.
```cpp
...
namespace tic_tac_toe {
   ...
   struct movement {
      uint32_t    row;
      uint32_t    column;

      EOSLIB_SERIALIZE( movement, (row)(column) )
   };

   struct move {
      account_name   challenger;
      account_name   host;
      account_name   by; // the account who wants to make the move
      movement       mvt;

      EOSLIB_SERIALIZE( move, (challenger)(host)(by)(mvt) )
   };
   ...
}
```
완성된 tic_tac_toe.hpp 파일은 [여기](https://github.com/EOSIO/eos/blob/master/contracts/tic_tac_toe/tic_tac_toe.hpp) 서 확인하실 수 있습니다.

### Main
tic_tac_toe.cpp 파일을 열고 아래의 boilerplate 처럼 시작하세요.
```cpp
#include "tic_tac_toe.hpp"
using namespace eosio;
/**
*  The apply() method must have C calling convention so that the blockchain can lookup and
*  call these methods.
*/
extern "C" {

   using namespace tic_tac_toe;
   /// The apply method implements the dispatch of events to this contract
   void apply( uint64_t receiver, uint64_t code, uint64_t action ) {
      // Put your action handler here
   }

} // extern "C"
```

#### 액션 핸들러
tic_tac_toe 컨트랙트가 오직 `tic.tac.toe` 계정으로 전달되는 액션에만 반응하고, 액션의 타입에 따라 다르게 반응하기를 원합니다. 'on' 메소드를 오버로딩하여 다른 액션 타입들을 가질 impl 구조체를 추가해 보겠습니다(이 예제에서 이 과정은 오버스펙으로 보입니다만, 화폐 등의 다른 확장 가능한 컨트랙트에 이 패턴이 적용돼 있는 경우를 종종 보시게 될 겁니다).
```cpp
using namespace eosio;
namespace tic_tac_toe {
struct impl {
   ...
   /// The apply method implements the dispatch of events to this contract
   void apply( uint64_t receiver, uint64_t code, uint64_t action ) {

      if (code == code_account) {
         if (action == N(create)) {
            impl::on(eosio::unpack_action_data<tic_tac_toe::create>());
         } else if (action == N(restart)) {
            impl::on(eosio::unpack_action_data<tic_tac_toe::restart>());
         } else if (action == N(close)) {
            impl::on(eosio::unpack_action_data<tic_tac_toe::close>());
         } else if (action == N(move)) {
            impl::on(eosio::unpack_action_data<tic_tac_toe::move>());
         }
      }
   }

};
}

...
extern "C" {

   using namespace tic_tac_toe;
   /// The apply method implements the dispatch of events to this contract
   void apply( uint64_t receiver, uint64_t code, uint64_t action ) {
      impl().apply(receiver, code, action);
   }

} // extern "C"
```

먼저 `unpack_action_data<T>()`를 특정 핸들러에 넘기기 전에 사용하고 있는 점에 주목하세요, `unpack_action_data<T>()`는 컨트랙트가 받은 액션을 `struct T`로 변환합니다.

이 과정을 깔끔하게 하기 위해, 액션 핸들러를 `struct impl` 내로 캡슐화했습니다:
```cpp
...
struct impl {
...
   /**
    * @param create - action to be applied
    */
   void on(const create& c) {
      // Put code for create action here
   }

   /**
    * @brief Apply restart action
    * @param restart - action to be applied
    */
   void on(const restart& r) {
      // Put code for restart action here
   }

   /**
    * @brief Apply close action
    * @param close - action to be applied
    */
   void on(const close& c) {
      // Put code for close action here
   }

   /**
    * @brief Apply move action
    * @param move - action to be applied
    */
   void on(const move& m) {
      // Put code for move action here
   }

   /// The apply method implements the dispatch of events to this contract
   void apply( uint64_t receiver, uint64_t code, uint64_t action ) {
...
```

#### Create 액션 핸들러
Create 액션 핸들러에는 다음이 필요합니다:
1. 액션에 호스트의 서명이 있는지 보장합니다
2. 호스트와 도전자가 같지 않다는 것을 보장합니다
3. 진행중인 게임이 없다는 것을 보장합니다
4. 새로 시작된 게임을 DB에 저장합니다
```cpp
struct impl {
   ...
   /**
    * @brief Apply create action
    * @param create - action to be applied
    */
   void on(const create& c) {
      require_auth(c.host);
      eosio_assert(c.challenger != c.host, "challenger shouldn't be the same as host");

      // Check if game already exists
      games existing_host_games(code_account, c.host);
      auto itr = existing_host_games.find( c.challenger );
      eosio_assert(itr == existing_host_games.end(), "game already exists");

      existing_host_games.emplace(c.host, [&]( auto& g ) {
         g.challenger = c.challenger;
         g.host = c.host;
         g.turn = c.host;
      });
   }
   ...
}

```

#### Restart 액션 핸들러
Restart 액션 핸들러에는 다음이 필요합니다:
1. 액션에 호스트나 도전자의 서명이 있는지 보장합니다
2. 진행중인 게임이 있다는 것을 보장합니다
3. Restart 액션이 호스트나 도전자에게서 수행되는지 보장합니다
4. 게임을 리셋합니다
5. 변경된 게임 상태를 DB에 저장합니다
```cpp
struct impl {
   ...
   /**
    * @brief Apply restart action
    * @param restart - action to be applied
    */
   void on(const restart& r) {
      require_auth(r.by);

      // Check if game exists
      games existing_host_games(code_account, r.host);
      auto itr = existing_host_games.find( r.challenger );
      eosio_assert(itr != existing_host_games.end(), "game doesn't exists");

      // Check if this game belongs to the action sender
      eosio_assert(r.by == itr->host || r.by == itr->challenger, "this is not your game!");

      // Reset game
      existing_host_games.modify(itr, itr->host, []( auto& g ) {
         g.reset_game();
      });
   }
   ...
}

```

#### Close 액션 핸들러
Close 액션 핸들러에는 다음이 필요합니다:
1. 액션에 호스트의 서명이 있는지 보장합니다
2. 진행중인 게임이 있다는 것을 보장합니다
3. 게임을 DB에서 삭제합니다
```cpp
struct impl {
   ...
   /**
    * @brief Apply close action
    * @param close - action to be applied
    */
   void on(const close& c) {
      require_auth(c.host);

      // Check if game exists
      games existing_host_games(code_account, c.host);
      auto itr = existing_host_games.find( c.challenger );
      eosio_assert(itr != existing_host_games.end(), "game doesn't exists");

      // Remove game
      existing_host_games.erase(itr);
   }
   ...
}

```

#### Move 액션 핸들러
Move 액션 핸들러에는 다음이 필요합니다:
1. 액션에 호스트나 도전자의 서명이 있는지 보장합니다
2. 진행중인 게임이 있는지 보장합니다
3. 게임이 아직 끝나지 않았음을 보장합니다
4. Move 액션이 호스트나 도전자에게서 수행되는지 보장합니다
5. 맞는 사용자 차례인지 보장합니다
6. 맞는 행동인지 보장합니다
7. 새 행동을 반영해 판을 변경합니다
8. move_turn을 다른 플레이어로 변경합니다
9. 승자가 있는지 확인합니다
10. 변경된 게임 상태를 DB에 저장합니다
```cpp
struct impl {
   ...
   bool is_valid_movement(const movement& mvt, const game& game_for_movement) {
      // Put code here
   }

   account_name get_winner(const game& current_game) {
      // Put code here
   }
   ...
   /**
    * @brief Apply move action
    * @param move - action to be applied
    */
   void on(const move& m) {
      require_auth(m.by);

      // Check if game exists
      games existing_host_games(code_account, m.host);
      auto itr = existing_host_games.find( m.challenger );
      eosio_assert(itr != existing_host_games.end(), "game doesn't exists");

      // Check if this game hasn't ended yet
      eosio_assert(itr->winner == N(none), "the game has ended!");
      // Check if this game belongs to the action sender
      eosio_assert(m.by == itr->host || m.by == itr->challenger, "this is not your game!");
      // Check if this is the  action sender's turn
      eosio_assert(m.by == itr->turn, "it's not your turn yet!");


      // Check if user makes a valid movement
      eosio_assert(is_valid_movement(m.mvt, *itr), "not a valid movement!");

      // Fill the cell, 1 for host, 2 for challenger
      const auto cell_value = itr->turn == itr->host ? 1 : 2;
      const auto turn = itr->turn == itr->host ? itr->challenger : itr->host;
      existing_host_games.modify(itr, itr->host, [&]( auto& g ) {
         g.board[m.mvt.row * 3 + m.mvt.column] = cell_value;
         g.turn = turn;

         //check to see if we have a winner
         g.winner = get_winner(g);
      });
   }
   ...
}

```
#### 행동 검증
빈 칸에 행동하는 것을 맞는 행동으로 정의합니다:
```cpp
struct impl {
   ...
   /**
    * @brief Check if cell is empty
    * @param cell - value of the cell (should be either 0, 1, or 2)
    * @return true if cell is empty
    */
   bool is_empty_cell(const uint8_t& cell) {
      return cell == 0;
   }

   /**
    * @brief Check for valid movement
    * @detail Movement is considered valid if it is inside the board and done on empty cell
    * @param movement - the movement made by the player
    * @param game - the game on which the movement is being made
    * @return true if movement is valid
    */
   bool is_valid_movement(const movement& mvt, const game& game_for_movement) {
      uint32_t movement_location = mvt.row * 3 + mvt.column;
      bool is_valid = movement_location < board_len && is_empty_cell(game_for_movement.board[movement_location]);
      return is_valid;
   }
   ...
}
```
#### 승자 결정
먼저 가로나 세로, 대각선으로 3개의 자기 표시(`1` 혹은 `2`)를 놓은 플레이어를 승자로 정의합니다.
```cpp
struct impl {
   ...
   /**
    * @brief Get winner of the game
    * @detail Winner of the game is the first player who made three consecutive aligned movement
    * @param game - the game which we want to determine the winner of
    * @return winner of the game (can be either none/ draw/ account name of host/ account name of challenger)
    */
   account_name get_winner(const game& current_game) {
      if((current_game.board[0] == current_game.board[4] && current_game.board[4] == current_game.board[8]) ||
         (current_game.board[1] == current_game.board[4] && current_game.board[4] == current_game.board[7]) ||
         (current_game.board[2] == current_game.board[4] && current_game.board[4] == current_game.board[6]) ||
         (current_game.board[3] == current_game.board[4] && current_game.board[4] == current_game.board[5])) {
         //  - | - | x    x | - | -    - | - | -    - | x | -
         //  - | x | -    - | x | -    x | x | x    - | x | -
         //  x | - | -    - | - | x    - | - | -    - | x | -
         if (current_game.board[4] == 1) {
            return current_game.host;
         } else if (current_game.board[4] == 2) {
            return current_game.challenger;
         }
      } else if ((current_game.board[0] == current_game.board[1] && current_game.board[1] == current_game.board[2]) ||
                 (current_game.board[0] == current_game.board[3] && current_game.board[3] == current_game.board[6])) {
         //  x | x | x       x | - | -
         //  - | - | -       x | - | -
         //  - | - | -       x | - | -
         if (current_game.board[0] == 1) {
            return current_game.host;
         } else if (current_game.board[0] == 2) {
            return current_game.challenger;
         }
      } else if ((current_game.board[2] == current_game.board[5] && current_game.board[5] == current_game.board[8]) ||
                 (current_game.board[6] == current_game.board[7] && current_game.board[7] == current_game.board[8])) {
         //  - | - | -       - | - | x
         //  - | - | -       - | - | x
         //  x | x | x       - | - | x
         if (current_game.board[8] == 1) {
            return current_game.host;
         } else if (current_game.board[8] == 2) {
            return current_game.challenger;
         }
      } else {
         bool is_board_full = true;
         for (uint8_t i = 0; i < board_len; i++) {
            if (is_empty_cell(current_game.board[i])) {
               is_board_full = false;
               break;
            }
         }
         if (is_board_full) {
            return N(draw);
         }
      }
      return N(none);
   }
   ...
}
```
완성된 tic_tac_toe.cpp 파일은 [여기](https://github.com/EOSIO/eos/blob/master/contracts/tic_tac_toe/tic_tac_toe.cpp)에서 확인하실 수 있습니다.

### ABI 만들기
ABI(Application Binary Interface)가 있어야만 컨트랙트가 바이너리로 전송한 액션을 이해할 수 있습니다.
tic_tac_toe.abi 파일을 열고 아래의 boilerplate 처럼 시작하세요:
```json
{
  "types": [],
  "structs": [{
      "name": "...",
      "base": "...",
      "fields": { ... }
  }, ...],
  "actions": [{
      "name": "...",
      "type": "...",
      "ricardian_contract": "..."
  }, ...],
  "tables": [{
      "name": "...",
      "type": "...",
      "index_type": "...",
      "key_names" : [...],
      "key_types" : [...]
  }, ...],
  "clauses: [...]
```
- types: 다른 자료구조체나 내장 자료형으로 표현되는 자료형의 리스트(c/c++의 typedef 같은)
- structs: 컨트랙트에서 사용되는 액션이나 테이블 자료 구조형의 리스트
- actions: 컨트랙트에서 사용할 수 있는 액션의 리스트
- tables: 컨트랙트에서 사용할 수 있는 테이블의 리스트

#### Table ABI
tic_tac_toe.hpp 파일에서는 games라는 이름으로 단일 인덱스를 가지는 i64 테이블을 만들 것입니다. 여기에는 `game` 자료구조가 저장되며, `challenger`가 키입니다. `challenger`의 자료형은 `account_name`이기 때문에 abi는 다음과 같게 됩니다:

```json
{
  ...
  "structs": [{
      "name": "game",
      "base": "",
      "fields": [
        {"name":"challenger", "type":"account_name"},
        {"name":"host", "type":"account_name"},
        {"name":"turn", "type":"account_name"},
        {"name":"winner", "type":"account_name"},
        {"name":"board", "type":"uint8[]"}
      ]
    },...
  ],
  "tables": [{
        "name": "games",
        "type": "game",
        "index_type": "i64",
        "key_names" : ["challenger"],
        "key_types" : ["account_name"]
      }
  ],
  ...
}
```

#### Actions ABI
액션을 위해 액션들을 `actions` 내부에 정의하고, 액션의 구조체를 `structs` 내부에 정의합니다.
```json
{
  ...
  "structs": [{
    ...
    },{
      "name": "create",
      "base": "",
      "fields": [
        {"name":"challenger", "type":"account_name"},
        {"name":"host", "type":"account_name"}
      ]
    },{
      "name": "restart",
      "base": "",
      "fields": [
        {"name":"challenger", "type":"account_name"},
        {"name":"host", "type":"account_name"},
        {"name":"by", "type":"account_name"}
      ]
    },{
      "name": "close",
      "base": "",
      "fields": [
        {"name":"challenger", "type":"account_name"},
        {"name":"host", "type":"account_name"}
      ]
    },{
      "name": "movement",
      "base": "",
      "fields": [
        {"name":"row", "type":"uint32"},
        {"name":"column", "type":"uint32"}
      ]
    },{
      "name": "move",
      "base": "",
      "fields": [
        {"name":"challenger", "type":"account_name"},
        {"name":"host", "type":"account_name"},
        {"name":"by", "type":"account_name"},
        {"name":"mvt", "type":"movement"}
      ]
    }
  ],
  "actions": [{
      "name": "create",
      "type": "create",
      "ricardian_contract": ""
    },{
      "name": "restart",
      "type": "restart",
      "ricardian_contract": ""
    },{
      "name": "close",
      "type": "close",
      "ricardian_contract": ""
    },{
      "name": "move",
      "type": "move",
      "ricardian_contract": ""
    }
  ],
  ...
}
```

### 컴파일!
이제 tic_tac_toe.cpp 파일을 컴파일해서 만든 tic_tac_toe.wast 파일을 이용해 컨트랙트를 nodeos에 배포할 것입니다.
```bash
$ eosiocpp -o tic_tac_toe.wast tic_tac_toe.cpp
```

### 배포!
이제 wast와 abi 파일(tic_tac_toe.wast, tic_tac_toe.abi)이 준비됐습니다. 배포의 시간입니다!
디렉토리(편의상 tic_tac_toe로 합시다)를 만들고 생성된 tic_tac_toe.wast, tic_tac_toe.abi 파일을 복사하세요.

```bash
$ cleos set contract tic.tac.toe tic_tac_toe
```
지갑이 잠금 해제되었고, `tic.tac.toe` 키가 임포트돼 있는지 확인하세요. 컨트랙트를 `tic.tac.toe`가 아닌 다른 계정으로 업로드하시려면, `tic.tac.toe` 부분을 여러분이 사용하실 계정명으로 바꾸고, 지갑에 해당 계정에 대한 키가 있는지 확인하세요.

### 플레이!
배포가 끝나 트랜잭션이 확정되면, 블록체인에서 컨트랙트를 쓸 수 있게 된 것입니다. 이제 플레이할 수 있습니다!

#### Create
```bash
$ cleos push action tic.tac.toe create '{"challenger":"inita", "host":"initb"}' --permission initb@active 
```
#### Move
```bash
$ cleos push action tic.tac.toe move '{"challenger":"inita", "host":"initb", "by":"initb", "mvt":{"row":0, "column":0} }' --permission initb@active
$ cleos push action tic.tac.toe move '{"challenger":"inita", "host":"initb", "by":"inita", "mvt":{"row":1, "column":1} }' --permission inita@active
```
#### Restart
```bash
$ cleos push action tic.tac.toe restart '{"challenger":"inita", "host":"initb", "by":"initb"}' --permission initb@active 
```
#### Close
```bash
$ cleos push action tic.tac.toe close '{"challenger":"inita", "host":"initb"}' --permission initb@active
```
#### 게임 상태 확인
```
$ cleos get table tic.tac.toe initb games
{
  "rows": [{
      "challenger": "inita",
      "host": "initb",
      "turn": "inita",
      "winner": "none",
      "board": [
        1,
        0,
        0,
        0,
        2,
        0,
        0,
        0,
        0
      ]
    }
  ],
  "more": false
}
```

## 번역 정보

* 위키 원문 : https://github.com/EOSIO/eos/wiki/Tutorial-Tic-Tac-Toe
* 번역 기준 리비전 : 0c18223ea118597c7b2476118091c4094be6af99
