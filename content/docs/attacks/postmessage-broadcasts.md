+++
title = "postMessage Broadcasts"
description = ""
date = "2020-10-01"
category = [
    "Attack",
]
abuse = [
    "postMessage",
]
defenses = [
    "Application Fix",
]
menu = "main"
weight = 3
+++

�A�v���P�[�V�����́A���̃I���W���Ə������L���邽�߂ɁA���΂���[postMessage broadcasts](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage)���g�p����B
`postMessage` ���g���ƁA2��ނ� XS-Leaks �ɂȂ���\��������B

* �M���ł��Ȃ����M���Ƌ@�����̍������b�Z�[�W�����L���邱��
    * `postMessage` API �� `targetOrigin` �p�����[�^���T�|�[�g���Ă���A������g�p���ă��b�Z�[�W����M�ł���I���W���𐧌����邱�Ƃ��ł���B���b�Z�[�W�ɋ@�����̍����f�[�^���܂܂�Ă���ꍇ�A���̃p�����[�^���g�p���邱�Ƃ��d�v�ł���B 

* �ω�����R���e���c��u���[�h�L���X�g�̑��݂Ɋ�Â������̃��[�N
    * ����XS-Leak�̃e�N�j�b�N�Ɠ��l�ɁA����̓I���N�����`�����邽�߂Ɏg����\��������B�Ⴆ�΁A����A�v���P�[�V�������A�^����ꂽ���[�U���������[�U�����݂���ꍇ�ɂ̂݁A�uPage Loaded�v�Ƃ��� postMessage �u���[�h�L���X�g�𑗐M����ƁA����𗘗p���ď������[�N���邱�Ƃ��ł���B
    
## �΍�

����XS-Leak�́ApostMessage�̃u���[�h�L���X�g���M�̖ړI�ɐ[���ˑ����邽�߁A���m�ȉ�����͑��݂��Ȃ��B
�A�v���P�[�V�����́ApostMessage�̒ʐM�����m�̑��M���O���[�v�ɐ�������K�v������B
���ꂪ�s�\�ȏꍇ�A�ʐM�̓��[�U�̏�ԂɊ֌W�Ȃ���т��ē�����������A�U���҂��ʐM�Ԃ̈Ⴂ�Ɋ�Â��ď��𐄘_����̂�h���ׂ��ł���B


## References

[^1]: Cross-Origin State Inference (COSI) Attacks: Leaking Web Site States through XS-Leaks, [link](https://arxiv.org/pdf/1908.02204.pdf)
