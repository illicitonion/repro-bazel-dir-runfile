sh_test(
    name = "test",
    srcs = ["test.sh"],
    args = ["$(location //dir:child)"],
    data = [
        "//dir:child",
        "//dir:runfiles",
    ],
)
