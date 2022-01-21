<h1 align="center">Merkle Tree in Golang</h1>
<p align="center">
<a href="https://travis-ci.org/cbergoon/merkletree"><img src="https://travis-ci.org/cbergoon/merkletree.svg?branch=master" alt="Build"></a>
<a href="https://goreportcard.com/report/github.com/cbergoon/merkletree"><img src="https://goreportcard.com/badge/github.com/cbergoon/merkletree?1=1" alt="Report"></a>
<a href="https://godoc.org/github.com/cbergoon/merkletree"><img src="https://img.shields.io/badge/godoc-reference-brightgreen.svg" alt="Docs"></a>
<a href="#"><img src="https://img.shields.io/badge/version-0.1.0-brightgreen.svg" alt="Version"></a>
</p>

An implementation of a Merkle Tree written in Go. A Merkle Tree is a hash tree that provides an efficient way to verify
the contents of a set data are present and untampered with.

At its core, a Merkle Tree is a list of items representing the data that should be verified. Each of these items
is inserted into a leaf node and a tree of hashes is constructed bottom up using a hash of the nodes left and
right children's hashes. This means that the root node will effictively be a hash of all other nodes (hashes) in
the tree. This property allows the tree to be reproduced and thus verified by on the hash of the root node
of the tree. The benefit of the tree structure is verifying any single content entry in the tree will require only
nlog2(n) steps in the worst case.

#### Documentation 

See the docs [here](https://godoc.org/github.com/cbergoon/merkletree).

#### Install
```
go get github.com/cbergoon/merkletree
```

#### Example Usage
Below is an example that makes use of the entire API - its quite small.
```go
package main

import (
  "crypto/sha256"
  "log"

  "github.com/cbergoon/merkletree"
)

//TestContent implements the Content interface provided by merkletree and represents the content stored in the tree.
type TestContent struct {
  x string
}

//CalculateHash hashes the values of a TestContent
func (t TestContent) CalculateHash() ([]byte, error) {
  h := sha256.New()
  if _, err := h.Write([]byte(t.x)); err != nil {
    return nil, err
  }

  return h.Sum(nil), nil
}

//Equals tests for equality of two Contents
func (t TestContent) Equals(other merkletree.Content) (bool, error) {
  return t.x == other.(TestContent).x, nil
}

func main() {
  //Build list of Content to build tree
  var list []merkletree.Content
  list = append(list, TestContent{x: "Hello"})
  list = append(list, TestContent{x: "Hi"})
  list = append(list, TestContent{x: "Hey"})
  list = append(list, TestContent{x: "Hola"})

  //Create a new Merkle Tree from the list of Content
  t, err := merkletree.NewTree(list)
  if err != nil {
    log.Fatal(err)
  }

  //Get the Merkle Root of the tree
  mr := t.MerkleRoot()
  log.Println(mr)

  //Verify the entire tree (hashes for each node) is valid
  vt, err := t.VerifyTree()
  if err != nil {
    log.Fatal(err)
  }
  log.Println("Verify Tree: ", vt)

  //Verify a specific content in in the tree
  vc, err := t.VerifyContent(list[0])
  if err != nil {
    log.Fatal(err)
  }

  log.Println("Verify Content: ", vc)

  //String representation
  log.Println(t)
}

```
#### Example for Keccak256
```go
type Keccak256Content struct {
	x string
}

func has0xPrefix(str string) bool {
	return len(str) >= 2 && str[0] == '0' && (str[1] == 'x' || str[1] == 'X')
}

//CalculateHash hashes the values of a TestContent
func (t Keccak256Content) CalculateHash() ([]byte, error) {
	s := t.x
	if has0xPrefix(s) {
		s = s[2:]
	}
	if len(s)%2 == 1 {
		s = "0" + s
	}
	h, _ := hex.DecodeString(s)
	k := keccak256.New()
	return k.Hash(h), nil
}

//Equals tests for equality of two Contents
func (t Keccak256Content) Equals(other Content) (bool, error) {
	return t.x == other.(Keccak256Content).x, nil
}

func main() {
    var list []Content
    list = append(list, Keccak256Content{x: "0x5B38Da6a701c568545dCfcB03FcB875f56beddC4"})
    list = append(list, Keccak256Content{x: "0xAb8483F64d9C6d1EcF9b849Ae677dD3315835cb2"})
    list = append(list, Keccak256Content{x: "0x4B20993Bc481177ec7E8f571ceCaE8A9e22C02db"})
    list = append(list, Keccak256Content{x: "0x78731D3Ca6b7E34aC0F824c42a7cC18A495cabaB"})
    list = append(list, Keccak256Content{x: "0x617F2E2fD72FD9D5503197092aC168c91465E7f2"})
    list = append(list, Keccak256Content{x: "0x17F6AD8Ef982297579C203069C1DbfFE4348c372"})
    list = append(list, Keccak256Content{x: "0x5c6B0f7Bf3E7ce046039Bd8FABdfD3f9F5021678"})
    //list = append(list, Keccak256Content{x: "0x5c6B0f7Bf3E7ce046039Bd8FABdfD3f9F5021678"})
    
    //Create a new Merkle Tree from the list of Content
    tree, err := NewTree(list, true)
    if err != nil {
        log.Fatal(err)
    }
    
    //Get the Merkle Root of the tree
    mr := tree.MerkleRoot()
    log.Println(mr)
    fmt.Println(hex.EncodeToString(mr))
    
    //Verify the entire tree (hashes for each node) is valid
    vt, err := tree.VerifyTree()
    if err != nil {
        log.Fatal(err)
    }
    log.Println("Verify Tree: ", vt)
    
    //Verify a specific content in in the tree
    vc, err := tree.VerifyContent(list[0])
    if err != nil {
        log.Fatal(err)
    }
    
    log.Println("Verify Content: ", vc)
    
    //String representation
    log.Println(tree)
    proof, _, _ := tree.GetMerklePath(list[6])
    for _, v := range proof {
        fmt.Println(hex.EncodeToString(v))
    }
}
```

#### Sample
![merkletree](merkle_tree.png)


#### License
This project is licensed under the MIT License.
