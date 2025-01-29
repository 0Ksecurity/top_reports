# Introduction

Last year, Immunefi hosted its first and most exciting Attackathon on their platform: the Fuel Attackathon, with a total bounty of $1 million. The Attackathon was divided into three categories based on programming languages: Rust, Sway, Solidity, TypeScript.

This event marked my first participation in a major contest/Attackathon as a security researcher. During this journey, I began learning about the Fuel blockchain and, most importantly, the Sway programming language. At the time, I had no prior experience with Rust-like languages or non-EVM blockchains. Despite this, I successfully secured 5th place in the Attackathon. In this Attackathon,

I focused full time on the Sway language category, which included exploring the Sway standard libraries, Sway libraries and standards, and Sway tooling.

This report highlights how I discovered one of the vulnerabilities that earned me $33.6k out of a total of $86,000 in rewards.

# Disclaimer

The details shared in this report are for informational purposes only and are not intended to encourage or discourage users or investors from engaging with the mentioned bug bounty program. This report highlights a vulnerability I discovered while reviewing the specified protocol during a set period, offering insights from my personal experience. Please conduct your own research and due diligence before investing in or working on mentioned bounty/contest.

# About **FUEL blockchain**

Fuel is an operating system purpose built for Ethereum Rollups. Fuel allows rollups to solve for PSI (parallelization, state minimized execution, interoperability) without making any, for more information read their docs [here](https://docs.fuel.network/docs/intro/what-is-fuel/).

# Report Stats

| Field              | Details                                                                                      |
| ------------------ | -------------------------------------------------------------------------------------------- |
| Attackathon name   | [Fuel Network](https://immunefi.com/audit-competition/fuel-network-attackathon/leaderboard/) |
| Report state       | PAID                                                                                         |
| Report Severity    | HIGH                                                                                         |
| Report status      | Chief Finding                                                                                |
| total bugs found   | 3 High(3 cheif finder), 5 low/insight                                                        |
| amount payed       | $33.6k                                                                                       |
| protocol team rate | 5/5                                                                                          |

# Vulnerability Details

## [HIGH] double increasing underlying value in `ceil` function

### Description

During my deep dive into the **fixed_point** library, one of the most essential and widely used libraries for future applications, I discovered a critical logic issue in the `ceil` function. This function, which is also utilized in the `round` function, is designed to return the smallest value that is equal to or greater than the underlying value, much like the behavior of the ceil function in Solidity libraries.

However, there is a critical issue in the ceil function that causes the underlying value to be increased twice when `non_negative` is set to false. This happens because the function inadvertently adds 1 to the underlying value twice, leading to unintended behavior. This issue can have significant consequences in applications that rely on precise value rounding.

### in depth analysis

I identified a flaw in the implementation that caused the underlying value to double increment unexpectedly. Let’s take a closer look at the ceil function, which can be found [here](https://github.com/FuelLabs/sway-libs/blob/0f47d33d6e5da25f782fc117d4be15b7b12d291b/libs/src/fixed_point/ifp128.sw#L466-L488):

```sway
pub fn ceil(self) -> Self {
        let mut underlying = self.underlying;
        let mut non_negative = self.non_negative;

        if self.non_negative {
            underlying = self.underlying.ceil();
        } else {

            let ceil = self.underlying.ceil(); //@audit increase/round up

            if ceil != self.underlying {
                underlying = ceil + UFP64::from(1); // @audit round up again
                if ceil == UFP64::from(1) {
                    non_negative = true;
                }
            } else {
                underlying = ceil;
            }
        }
        Self {
            underlying: underlying,
            non_negative: self.non_negative,
        }
    }
```

When self.non_negative is false, the else clause of the function will execute. Upon close inspection of the function, we can observe that the underlying value is incremented due to the following line:

`let ceil = self.underlying.ceil();`

This line calls the `ceil` function on the underlying value which increasing it. Subsequently, the condition `if ceil != self.underlying` will evaluate to true. This happens because the ceil value no longer equals the self.underlying value since it was incremented in the previous step. As a result, the underlying value is incremented a second time, leading to a double increase in the returned underlying value. This value is later utilized by the `round` function.

The severity of this bug is classified as high because the `fixed_point ` library is a foundational utility that will likely be adopted by many protocols built on the Fuel blockchain. A double increment in the ceil function can result in unintended behaviors, such as theft of yield, logical errors, or other critical vulnerabilities in future protocols.

## Final words

I’m proud to have dedicated 17–20 days to working on the Fuel Attackathon. During this time, I not only learned a completely new programming language in a short period but also secured 5th place on the Attackathon leaderboard. It was a challenging yet rewarding experience that expanded my skills and knowledge significantly.
