## Wed Jun 29

I'm in love.  🎧 [Dua Lipa](https://open.spotify.com/track/0LnS7aOdOdI1dNKZqdOLz4?si=9e7a8fa5c3974ee4)


---



Last week in NYC was crazzie. I’m still having a pretty terrible hangover (covid or whatever). <br /> <br />

Around a week ago, I saw this https://twitter.com/1zaqk1/status/1538646652998778882 and curious how a while-like for loop is a way cheaper than a known-way for loop  


<br />
<br />


```jsx


// SPDX-License-Identifier: MIT

pragma solidity ^0.8.7;


contract forLoops {

    function inc0(uint ep) external pure returns (uint) {
        uint value;
        for (uint i; i <= ep; i++) {
            value = i;
        }
        unchecked { return value; }
    }


    function inc1(uint ep) external pure returns (uint) {
        uint value;
        for (uint i; i < ep;) {
            unchecked { ++i; }
            value = i;
        }
        return value;
    }


    function inc2(uint ep) external pure returns (uint) {
        uint value;
        for (uint i;;) {
            unchecked { i++; }
            value = i;
            if (i >= ep) break;
        }
        return value;
    }


    function inc3(uint ep) external pure returns (uint) {
        uint value;
        while (value < ep) {
            value ++;
        }
        return value;
    } 


    function inc4(uint ep) external pure returns (uint) {
        uint value;
        do {
            value ++;
        } while (value < ep);
        return value;
    }
}
```

<br />
<br />



| ep     | 10          | 100  | 1000 |
| ------------- |:-------------:| -----:| ---- |
| inc0 (for / unchecked { return value; }   | 23864	| 40424	| 206036 |
| inc1 (for / unchecked { ++i; } )      | 22455 |	27765	|80877|
| inc2 (for / ;; and break)| 	22414	|27994	|83806|
| inc3 (while)| 23694	|39804	|200916|
| inc4 (do-while)|23505|	38355	|186867|



### WIP
